# Security audits and bug hunting findings by Rajkumar

This repository represents my collection of bug hunting findings for my portfolio.

| Vulnerability                                                                                                                                             | Severity      | Protocol     | Platform |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------ | -------- |
| [Wrong interest rate calculation](Immunefi/README.md#wrong-interest-rate-calculation)                                                                     | High          | UniLend      | Immunefi |
| [Bypassing modify Blacklist function](Immunefi/README.md#bypassing-modify-blacklist-function)                                                             | Medium        | Aura Finance | Immunefi |
| [Owner can steal all user funds](Immunefi/README.md#owner-can-steal-all-user-funds)                                                                       | Medium        | Davos        | Immunefi |
| [lend() function always return minted tokens equal to zero](Immunefi/README.md#lend-function-always-return-minted-tokens-equal-to-zero)                   | Low           | UniLend      | Immunefi |
| [Wrong use of assembly builtin function](Immunefi/README.md#wrong-use-of-assembly-builtin-function)                                                       | Low           | Hyperlane    | Immunefi |
| [createCanonicalERC20Wrapper reverts on right erc20 implementation](Immunefi/README.md#createcanonicalerc20wrapper-reverts-on-right-erc20-implementation) | Low           | Superfluid   | Immunefi |
| [Unchecked low level call](Immunefi/README.md#unchecked-low-level-call)                                                                                   | Low           | Aurora       | Immunefi |
| [Wrong emission of event](Immunefi/README.md#wrong-emission-of-event)                                                                                     | Informational | Revest       | Immunefi |
| [Wrong implementation of supportsInterface()](Immunefi/README.md#wrong-implementation-of-supportsinterface)                                               | Informational | Revest       | Immunefi |

## Contacts

I am available for smart contract security consulting. Reach out to me on:

- Twitter - [@0xRajkumar](https://twitter.com/0xRajkumar)
- Discord - [0xRajkumar#1861](https://discordapp.com/users/794774992964550656)
