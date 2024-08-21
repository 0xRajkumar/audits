# Security Review of Art.Tech

A time-boxed security review of **Art.Tech** system was conducted by **Rajkumar**, focusing on the security aspects of the smart contract implementation.

Disclaimer: A security review is a time-bound effort aimed at providing the highest value possible within the available time. As such, it does not guarantee the absence of vulnerabilities. Additional security reviews, a bug bounty program, and on-chain monitoring are recommended.

## About Rajkumar

[Rajkumar](https://twitter.com/0xRajkumar) is an independent security researcher with extensive experience in smart contract audits and bug hunting.

# Overview of Art.Tech

Art.Tech is a platform that infuses Generative AI with blockchain. Users can mint an artwork as an ERC1155 using AI with a single click. Each trade contributes to the prize pool, the creator and the protocol in fees; creators and collectors compete for prizes. 


## Scope

https://gist.github.com/0xRajkumar/f219d8adeacbbca80a7cf58ae9d21ff5

# Findings Summary

| Title                                                                                                                                                     | Severity      | Status      | 
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |  -------------      | 
| [H-01] Permanent DOS in Buy and Sell art function because the Creator can revert when the Creator is a contract                                           | High          |  Resolved      | 
| [H-02] DoS when calling `pruneExpiredActions` with an expired action at index 0                                                                      | High          |  Resolved      | 
| [H-03] Loss of user funds when selling art due to front-running                                          | High          |  Resolved      | 
| [M-01] `pruneExpiredActions` should be called before checking `pendingActionIds.length < MAX_PENDING_ACTIONS` otherwise, it could cause a DoS.                                           | Medium          |  Resolved      | 
| [M-02] We should return any extra paid eth in buyArt function                                            | Medium          |  Resolved      | 
| [L-01] Emitting the wrong `endArtId` in the `RoundEnded` event                                           | Low          |  Resolved      | 
|[L-02] Emitting the wrong `pricePaid` in the `ArtMinted` event                                        | Low          |  Resolved      | 


# Severity Classification Framework

| Severity               | Impact: High | Impact: Medium | Impact: Low |
| ---------------------- | ------------ | -------------- | ----------- |
| **Likelihood: High**   |  High        | High           | Medium      |
| **Likelihood: Medium** | High         |  Medium        | Low         |
| **Likelihood: Low**    | Medium       |  Low           | Low        |

----------

#  [H-01] Permanent DOS in Buy and Sell art function because the Creator can revert when the Creator is a contract

## Description

Both the `buyArt` and `sellArt` functions send fees to the Creator using the `_sendFees` method. We check if the transfer is successful if it is, the execution continues, but if not, the process stops. This approach works well when the Creator is an EOA (Externally Owned Account). However, when the Creator is a contract, it can cause the buyArt and sellArt functions to revert.

Here’s a scenario: A user purchases art, but when they try to sell it, they won't be able to if the fee transfer fails. This could be due to a revert intentionally triggered in the contract's `fallback` or receive function.

### Severity

- Impact: High
- Likelihood: High

## Recommendations

Use Pull over Push pattern.

----------


#  [H-02] DoS when calling `pruneExpiredActions` with an expired action at index 0

## Description
In `pruneExpiredActions` we remove expired actions, but if there's an expired action at index 0, it causes a DoS.

This occurs because `i` is a uint256, and when we attempt `0 - 1`, it results in a revert.

As a result, we can't call this function to remove expired actions. 


```solidity
    function pruneExpiredActions() public {
        for (uint256 i = 0; i < pendingActionIds.length; i++) {
            bytes32 actionId = pendingActionIds[i];
            if (
                block.timestamp >= pendingAdminActions[actionId].expirationTime
            ) {
                removePendingAction(actionId);
                emit AdminActionPruned(actionId);
                i--; //HERE
            }
        }
    }
```

It also causes DoS in proposeAdminAction, since proposeAdminAction calls this function.

### Severity

- Impact: High
- Likelihood: High

## Recommendations

To mitigate we will only increment the i in else block.
```solidity
    function pruneExpiredActions() public {
        for (uint256 i = 0; i < pendingActionIds.length; ) {
            bytes32 actionId = pendingActionIds[i];
            if (
                block.timestamp >= pendingAdminActions[actionId].expirationTime
            ) {
                removePendingAction(actionId);
                emit AdminActionPruned(actionId);
            }else{
                i++;
            }
        }
    }
```

----------


#  [H-03] Loss of user funds when selling art due to front-running.

## Description
As we know, the price is determined by supply: if the supply is lower, the price is lower, and if the supply is higher, the price is higher. An attacker can exploit this through front-running.

Here’s a scenario to illustrate: A user wants to sell their art when the supply is 11, but the attacker front-runs the transaction and sells their art first. As a result, the supply decreases to 10 by the time the user's transaction is added to the block. The user was expecting the price at a supply of 11 but ended up with a lower price due to the supply being 10.


### Severity

- Impact: High
- Likelihood: High

## Recommendations
We need to introduce an additional parameter `minSupply` and check whether the supply is below this value. If the supply is less than `minSupply` the transaction will revert.

```solidity
    function sellArt(uint256 artId,uint256 minSupply, uint256 amount) external nonReentrant {
        require(totalSupply(artId) >= minSupply, "Less Supply");
        require(totalSupply(artId) > 1, "Cannot sell the last art");
        require(balanceOf(msg.sender, artId) >= amount, "Insufficient balance");

        uint256 grossPrice = getSellPrice(artId, amount);
        uint256 netPrice = getSellPriceAfterFees(artId, amount);
        uint256 creatorFee = (grossPrice * creatorEarningPercent) /
            feePrecision;
        uint256 prizePoolFee = (grossPrice * prizePoolRewardPercent) /
            feePrecision;
        uint256 protocolFee = (grossPrice * protocolFeePercent) / feePrecision;

        (bool success, ) = msg.sender.call{value: netPrice}("");
        require(success, "Unable to send funds to seller");

        _burn(msg.sender, artId, amount);
        _sendFees(creatorFee, prizePoolFee, protocolFee, artIdToCreator[artId]);
        lifetimeTradingVolume += grossPrice;
        currentRound.totalTradingVolume += grossPrice;

        emit ArtSold(
            artId,
            msg.sender,
            artIdToCreator[artId],
            currentRound.roundId,
            amount,
            grossPrice,
            netPrice,
            creatorFee,
            prizePoolFee,
            protocolFee
        );
        checkAndEndRound();
    }
```

----------

#  [M-01] `pruneExpiredActions` should be called before checking `pendingActionIds.length < MAX_PENDING_ACTIONS` otherwise, it could cause a DoS

## Description
In the `proposeAdminAction` function, we check if `pendingActionIds.length < MAX_PENDING_ACTIONS` If this condition is not met, we revert. However, we need to keep in mind that pendingActionIds may contain expired actions.

Currently, we call `pruneExpiredActions` to remove these expired actions after checking `pendingActionIds.length < MAX_PENDING_ACTIONS`. Instead, we should call pruneExpiredActions before checking `pendingActionIds.length < MAX_PENDING_ACTIONS` to avoid reverting due to expired actions.

### Severity

- Impact: Medium
- Likelihood: Medium

## Recommendations
My recommendation is that we should call `pruneExpiredActions` function before checking `pendingActionIds.length < MAX_PENDING_ACTIONS`.

----------

#  [M-02] We should return any extra paid eth in buyArt function

## Description
Scenario: A user wants to buy art when the supply is 11, but by the time the user's transaction is added to the block, the supply has decreased to 10. As a result, the price drops, and the user ends up paying extra ETH to mint the art. This extra ETH could get stuck in the contract if it's not refunded.

### Severity

- Impact: Medium
- Likelihood: Medium

## Recommendations
My recommendation in that we should return the extra eth paid.

----------

#  [L-01] Emitting the wrong `endArtId` in the `RoundEnded` event

## Description
In the `checkAndEndRound` function, setting `currentRound.endArtId` equal to `artIdCounter` is incorrect. This is because the round does not contain data related to `artIdCounter`, which represents art that has not yet been minted.


### Severity

- Impact: Low
- Likelihood: Medium

## Recommendations
We should set `currentRound.endArtId` equal to `artIdCounter - 1` to mitigate this issue.

----------

#  [L-02] Emitting the wrong `pricePaid` in the `ArtMinted` event

## Description
In the ArtMinted event, which is emitted when `mintArt` is called, the `pricePaid` can be zero if `addressToFreeMints` is greater than zero, or it can be the `mintFee` if not.

Currently, we check if `addressToFreeMints[msg.sender] == 0` is true, we emit mintFee; otherwise, we emit zero. This approach is incorrect because when `addressToFreeMints[msg.sender]` is 1, we did not charge `mintFee`, and we decreased `addressToFreeMints[msg.sender]` by 1, making it 0. Despite not charging the mintFee, we are incorrectly emitting it because `addressToFreeMints[msg.sender]` has transitioned from 1 to 0.

### Severity

- Impact: Low
- Likelihood: Medium

## Recommendations
We can use totalCost to mitigate this issue.
```solidity
    emit ArtMinted(
        newArtId,
        msg.sender,
        currentRound.roundId,
        totalCost
    );
```
----------