# Bugs founded on Immunefi

## Wrong interest rate calculation

#### Not Reported yet [see this](https://twitter.com/0xRajkumar/status/1641754411432132609)

```
    function getCurrentInterestRate(uint totalBorrow, uint availableBorrow) external pure returns (uint){
        uint uRate;
        if(totalBorrow > 0){
            uRate = (totalBorrow.mul(10**18)).div(availableBorrow.add(totalBorrow));
        }
        uint apy = uint(10).add( uRate.mul(30) );
        return apy.div(2102400); // per block interest
    }

```

Before we start 1 ether = 10\*\*18

Interest rate is equal to base rate which is 10% plus Utilization rate multiply by 30%. If Utilization is 0 then interest rate is 10% and if Utilization rate is 1 then interest rate is 40%.

As you can see our Utilization rate is multiple of 1 ether that's why during interest calculation principal is divided by extra 1 ether.

```
    function calculateInterest(uint _principal, uint _rate, uint _duration) internal pure returns (uint){
        return _principal.mul( _rate.mul(_duration) ).div(10**20);
    }
```

Dividing by 10\*20 because we have to divide principle by 1 ether plus divide by 100 to calculate interest

Bug: Here during APY calculation base interest rate is not multiplied by 1 ether that's why interest calculated is almost depends on dynamic rate

Example:

```
when utilization rate is 1 mean 1 ether

APY = 10 + (1 ether * 30)

Interest = principle*rate*duration/10**20 = principle*(10 + (1 ether * 30))*duration/10**20

Let's take 1 ether common from numerator

Interest = 1 ether *principle*(10/ 1 ether + 30)*duration/10**20 = principle*(10/ 1 ether + 30)*duration/10**2

```

After doing little math we can see base interest rate applied is 10/1 ether.

#### References 

https://github.com/UniLend/unilendv2/blob/main/contracts/pool.sol#L60
https://github.com/UniLend/unilendv2/blob/main/contracts/interestrate.sol#L14

#### Impact

High

## Bypassing modify Blacklist function.

Attacker can bypass this by running sandwich bot that looks for the modifyBlacklist() transaction, and if it sees it, it frontruns it with selfdestruct, and backruns it with create2 to get the contract again.

#### References

https://github.com/aurafinance/aura-contracts/blob/main/contracts/core/AuraLocker.sol#L203

#### Impact

Medium

## Owner can steal all user funds

Owner of MasterVault can easily perform reentrancy attack and steal all user funds

```
    function withdrawFee() external onlyOwner{
        if(feeEarned > 0) {
            IWETH(asset()).withdraw(feeEarned);
            feeReceiver.transfer(feeEarned);
            feeEarned = 0;
        }
    }

```

Q: How can it steal all tokens by re-entrancy when the .transfer method is used which forwards only 2300 gas to prevent re-entrancy?
A: I believe that if gas costs are subject to change in the future, the system may become vulnerable or not completely reentrancy-proof [see here](https://twitter.com/0xRajkumar/status/1641834656923373571).

#### References

https://github.com/davos-money/new-davos-smart-contracts/blob/main/contracts/MasterVault/MasterVault.sol#L474

#### Impact

Medium

Q: Why Medium?
A: Risk is high but at the same time difficulty to exploit is also hard that's why i have market it as Medium

##### Outcome

According to Immunefi it was technically valid but because of rule "Attacks requiring access to privileged addresses (governance, strategist)" it was out of scope

## lend() function always return minted tokens equal to zero.

The function lend() function of Pool.sol always returns zero minted Tokens. This creates issues with frontends that expect valid minted Tokens

#### References

https://github.com/UniLend/unilendv2/blob/09fc9be393684a3e4e3588da9d6dea20cac1211f/contracts/pool.sol#L532

#### Impact

Low

## Wrong use of assembly builtin function.

We have OwnableMulticall contract and Owner of OwnableMulticall can use proxyCalls.In this function we low level call which then returns bool and bytes of data(which is then stored in "returnData").

If bool is false mean call was not successfull then this uses assembly to revert and return "returnData"

Here assembly builtin revert function is used to do this work.  
Bug is that it takes two arguments add(returnData,32) and returnData from which second argument passed is wrong instead second argument should be return data size like "mload(returnData)" or "returndatasize()".

```
    function proxyCalls(Call[] calldata calls) external onlyOwner {
        for (uint256 i = 0; i < calls.length; i += 1) {
            (bool success, bytes memory returnData) = calls[i].to.call(
                calls[i].data
            );
            if (!success) {
                assembly {
                    revert(add(returnData, 32), returnData)
                }
            }
        }
    }

```

Because of this mistake it always returns wrong data when it reverts.

#### References

https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/1651c78ead3e2325dff5ddc50c64673eab15f5fd/solidity/contracts/OwnableMulticall.sol#L54

#### Impact

Low

## createCanonicalERC20Wrapper reverts on right erc20 implementation.

According to ERC20 standard name(), symbol() and decimals() are optionals and no one should expect these functions to be present in contract.

createCanonicalERC20Wrapper() of SuperTokenFactory calls these methods to get name, symbol and decimals and that's why if we pass correct ERC20 which do not implement these functions the call will fail and user will not be able to create canonical ERC20 wrapper of that ERC20 token.

We have DAI ERC20 token which is widely used and it returns name and symbol as bytes32 and decimals as uint256 that's why no one can create canonical ERC20 wrapper of DAI because our function expects name and symbol in string.

#### References

https://github.com/superfluid-finance/protocol-monorepo/blob/dev/packages/ethereum-contracts/contracts/superfluid/SuperTokenFactory.sol#L129

#### Impact

Low

## Unchecked low level call

In EvmErc20V2 and EvmErc20 we have withdrawToNear function and takes to parameters one is recipient(in bytes) and other one is amount. Function first uses \_burn() function to burn amount of user and uses assembly. In assembly it uses low call function and this function on fail do not revert instead returns 0 and it also not checked here and it can cause loss of user funds if user submits wrong recipient because function do not check if recipient length is equal to 20 Bytes.

#### References

https://github.com/aurora-is-near/aurora-engine/blob/master/etc/eth-contracts/contracts/EvmErc20.sol#L51

#### Impact

Low

## Wrong emission of event

In TokenVault contract we have withdrawToken() function which can be called by RevestController only because this function uses onlyRevestController modifier. This function also emits RedeemFNFT event with arguments fnftId(type is uint) and from(type is address) here "from" should be user who is redeeming tokens.

```
emit RedeemFNFT(fnftId, _msgSender());
```

Mistake is that it always passes msg.sender as "from" which is wrong because RevestController can only call it and that's why "from" will always be RevestController which is wrong. Instead it should pass user as "from" because user redeeming his token and correct code will be

```
emit RedeemFNFT(fnftId, user);
```

#### References

https://etherscan.io/address/0xA81bd16Aa6F6B25e66965A2f842e9C806c0AA11F?utm_source=immunefi#code

In TokenVault and line number is 121.

#### Impact

Informational

## Wrong implementation of supportsInterface().

Revest contract is using OpenZeppelin's ERC165Checker.sol contract of v4.4.1 version. ERC165Checker.sol implements supportsInterface() to check target contract supports interfaceId or not which is passed as argument to the function during call. The target contract of an EIP-165 supportsInterface query can cause unbounded gas consumption by returning a lot of data, while it is generally assumed that this operation has a bounded cost.

#### References

https://github.com/Revest-Finance/RevestContracts/blob/master/hardhat/contracts/Revest.sol#L137 https://github.com/Revest-Finance/RevestContracts/blob/master/hardhat/contracts/Revest.sol#L205 https://github.com/Revest-Finance/RevestContracts/blob/master/hardhat/contracts/Revest.sol#L258

#### Impact

Informational
