## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Similar code in `getCollateralValueInMalt` and `totalUsefulCollateral` functions ](https://github.com/code-423n4/2021-11-malt-findings/issues/180) 

# Handle

nathaniel


# Vulnerability details

## Impact
The code in `getCollateralValueInMalt` of ImpliedCollateralService.sol, can leverage the `totalUsefulCollateral` function, reducing code size and gas cost when calling the contract.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/ImpliedCollateralService.sol#L104-L124

## Tools Used
manual

## Recommended Mitigation Steps
Remove L108-L110, then in the return of `getCollateralValueInMalt` return `totalUsefulCollateral().mul(target).div(maltPrice) + swingTraderMaltBalance`

