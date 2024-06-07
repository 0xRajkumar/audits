# Security audits and bug hunting findings by Rajkumar

This repository represents my collection of bug hunting findings for my portfolio.

## For Immunefi findings

| Vulnerability                                                                                                                                             | Severity      | Protocol     | 
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------ | 
| [Wrong interest rate calculation](Immunefi/README.md#wrong-interest-rate-calculation)                                                                     | High          | UniLend      | 
| [Bypassing modify Blacklist function](Immunefi/README.md#bypassing-modify-blacklist-function)                                                             | Medium        | Aura Finance | 
| [Persistent DOS to stakeListing function](Immunefi/README.md#persistent-dOS-to-stakeListing-function)                                                     | Medium        | Arkham       | 
| [Owner can steal all user funds](Immunefi/README.md#owner-can-steal-all-user-funds)                                                                       | Medium        | Davos        | 
| [lend() function always return minted tokens equal to zero](Immunefi/README.md#lend-function-always-return-minted-tokens-equal-to-zero)                   | Low           | UniLend      | 
| [Wrong use of assembly builtin function](Immunefi/README.md#wrong-use-of-assembly-builtin-function)                                                       | Low           | Hyperlane    | 
| [Revert during calling claim function even when listing is closed](Immunefi/README.md#revert-during-calling-claim-function-even-when-listing-is-closed)   | Low           | Arkham       | 
| [createCanonicalERC20Wrapper reverts on right erc20 implementation](Immunefi/README.md#createcanonicalerc20wrapper-reverts-on-right-erc20-implementation) | Low           | Superfluid   | 
| [Unchecked low level call](Immunefi/README.md#unchecked-low-level-call)                                                                                   | Low           | Aurora       | 
| [Wrong emission of event](Immunefi/README.md#wrong-emission-of-event)                                                                                     | Informational | Revest       | 
| [Wrong implementation of supportsInterface()](Immunefi/README.md#wrong-implementation-of-supportsinterface)                                               | Informational | Revest       | 

## For Contest
| Protocol                                                                                                                                             | Findings      | Platform     | 
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------ | 
|  Goat Tech                                                                | 3H, 2M          | Cantina      | 
|  Asymmetry                                                                | 2H          | Code4rena      | 
|  Popcorn                                                                | 1H          | Code4rena      | 

## Contacts

I am available for smart contract security consulting. Reach out to me on:

- Twitter - [@0xRajkumar](https://twitter.com/0xRajkumar)
- Discord - [0xRajkumar#1861](https://discordapp.com/users/794774992964550656)
