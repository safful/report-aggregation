## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Incorrect implementation of arctan in the contract `FairSideFormula`](https://github.com/code-423n4/2021-05-fairside-findings/issues/73) 

# Handle

shw


# Vulnerability details

## Impact

The current implementation of the arctan formula in the contract `FairSideFormula` is inconsistent with the referenced paper and could cause incorrect results when the input parameter is negative. The erroneous formula affects the function `calculateDeltaOfFSD` and the number of FSD tokens minted or burned.

## Proof of Concept

The function `_arctan` misses two `abs` on the variable `a`. The correct implementation should be:

```solidity
function _arctan(bytes16 a) private pure returns (bytes16) {
    return
        a.mul(PI_4).sub(
            a.mul(a.abs().sub(ONE)).mul(APPROX_A.add(APPROX_B.mul(a.abs())))
        );
}
```

Notice that `_arctan` is called by `arctan`, and `arctan` is called by `arcs` with `ONE.sub(arcInner)` provided as the input parameter. Since `arcInner = MULTIPLIER_INNER_ARCTAN.mul(x).div(fS3_4)` can be a large number (recall that `x` is the capital pool), it is possible that the parameter `a` is negative.

Referenced code:

[FairSideFormula.sol#L45-L61](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/FairSideFormula.sol#L45-L61)
[FairSideFormula.sol#L77-L85](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/FairSideFormula.sol#L77-L85)
[FairSideFormula.sol#L127](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/dependencies/FairSideFormula.sol#L127)
[ABC.sol#L38](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/ABC.sol#L38)

## Recommended Mitigation Steps

Modify the `_arctan` function as above.

