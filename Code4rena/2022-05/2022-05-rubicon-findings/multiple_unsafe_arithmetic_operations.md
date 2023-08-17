## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Multiple Unsafe Arithmetic Operations](https://github.com/code-423n4/2022-05-rubicon-findings/issues/443) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L844
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L857
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L883
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L898
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L927
https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L951


# Vulnerability details

## RMT-02M: Multiple Unsafe Arithmetic Operations

| File | Lines | Type |
| :- | :- | :- |
| RubiconMarket.sol | [L844](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L844), [L857](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L857), [L883](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L883), [L898](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L898), [L927](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L927), [L951](https://github.com/code-423n4/2022-05-rubicon/blob/main/contracts/RubiconMarket.sol#L951) | Mathematical Operations |

### Description

The referenced lines all perform unsafe multiplications using the unitary denominations of either `1 ether` (`1e18`) or `10**9` (`1e9`), both of which can easily lead to overflows when used as a multiplier for large amounts of assets.

### Impact

Purchasing and selling amounts will be improperly fulfilled as well as improperly tracked as "sold out" / "bought out".

### Solution (Recommended Mitigation Steps)

We advise the codebase to make use of the `mul` operation exposed by the `DSMath` library already incorporated into the codebase to guarantee all operations are performed safely and cannot overflow.

### PoC

Issue is deducible by inspecting the relevant lines referenced in the issue and making note of the raw multiplication (`*`) operations performed.

### Tools

Manual inspection of the codebase.

