## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Duplicated code and usage of assert](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/96) 

# Handle

fr0zn


# Vulnerability details

## Vulnerability Details
The `_available_supply` and `available_supply` functions on the `AirdropDistribution` (lines [601](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L601) and [607](https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/AirdropDistribution.sol#L607)) do contain the exact same code.

Furthermore, the `assert` check inside those functions should be changed to a require statement since the check is not an invariant and gas refund should take place if the check fails ([SWC-110](https://swcregistry.io/docs/SWC-110)).

## Impact
Gas optimization

## Tools Used
Manual code review

## Recommended Mitigation Steps
The `available_supply` and `_available_supply` functions should be combined into a single public function (public functions can be used internally without any extra gas) or have the public function call the internal implementation and use the private implementation in the contract.

The `assert` check in the `available_supply` function should be changed to a require statement since the check is not an invariant.

