## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Use `call()` instead of `transfer()` when transferring ETH in RubiconRouter](https://github.com/code-423n4/2022-05-rubicon-findings/issues/82) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L356
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L374
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L434
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L451
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L491
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconRouter.sol#L548


# Vulnerability details

## Impact
When transferring ETH, use `call()` instead of `transfer()`.

The `transfer()` function only allows the recipient to use 2300 gas. If the recipient uses more than that, transfers will fail. In the future gas costs might change increasing the likelihood of that happening.

Keep in mind that `call()` introduces the risk of reentrancy. But, as long as the router follows the checks effects interactions pattern it should be fine. It's not supposed to hold any tokens anyway.

## Proof of Concept
See the linked code snippets above.

## Tools Used
none

## Recommended Mitigation Steps
Replace `transfer()` calls with `call()`. Keep in mind to check whether the call was successful by validating the return value:

```sol
(bool success, ) = msg.sender.call{value: amount}("");
require(success, "Transfer failed.")
```

