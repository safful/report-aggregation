## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [NamespaceCollision: Multiple SafeMath contracts](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/181) 

# Handle

heiho1


# Vulnerability details

## Impact

Got an error attempting Slither analysis due to several contracts defining "SafeMath"
  - In this case there are *two* distinct safe math libraries, one dependent on solc 0.6.11 and one dependent on solc ^0.8.0. This can lead to confusion during development.  Ideally the safe math version of the application contract [0.6.11] would be standardized but in this case *re-naming* the SafeMath contracts also suffices.

## Proof of Concept

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/AssetWrappers/WJLP/SafeMath.sol#L15

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/Dependencies/SafeMath.sol#L21



## Tools Used

Slither

## Recommended Mitigation Steps

Rename packages/contracts/contracts/AssetWrappers/WJLP/SafeMath.sol to SafeMath080.sol [and updating the declared contract to the same name]

