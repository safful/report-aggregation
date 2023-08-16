## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Adding Unchecked Directive will Save Gas for BurnMath.sol#getAsset and BurnMath.sol#getCollateral functions](https://github.com/code-423n4/2022-01-timeswap-findings/issues/183) 

# Handle

Rhynorater


# Vulnerability details

In `BurnMath.sol` we have the following function defined;
```
    function getAsset(IPair.State memory state, uint256 liquidityIn) internal pure returns (uint128 assetOut) {
        if (state.reserves.asset <= state.totalClaims.bond) return assetOut; 
        uint256 _assetOut = state.reserves.asset;
        _assetOut -= state.totalClaims.bond;
        _assetOut = _assetOut.mulDiv(liquidityIn, state.totalLiquidity);
        assetOut = _assetOut.toUint128();
}
```
Since the above `if` statement ensures that `state.reserves.asset` is  not less than or equal to `state.totalClaims.bond`, it is impossible for the `_assetOut -= state.totalClaims.bond; ` line to underflow. As a result, adding the `unchecked` directive around this will save on gas.

By the same reasoning, in the `getCollateral` function:
```
deficit -= state.reserves.asset;
```
is already checked by the 
```
if (state.reserves.asset >= state.totalClaims.bond) {
```
`if` statement. Surrounding this with `unchecked` will also save on gas.

Lastly, this also applies in `WithdrawMath.sol#getCollateral`:
```
        if (state.reserves.asset >= state.totalClaims.bond) return collateralOut;
        uint256 deficit = state.totalClaims.bond;
        deficit -= state.reserves.asset;
```
The deficit will never underflow here, so adding `unchecked` will save on gas. 

##References
https://github.com/code-423n4/2022-01-timeswap/blob/5960e07d39f2b4a60cfabde1bd51f4b1e62e7e85/Timeswap/Timeswap-V1-Core/contracts/libraries/BurnMath.sol#L22
https://github.com/code-423n4/2022-01-timeswap/blob/5960e07d39f2b4a60cfabde1bd51f4b1e62e7e85/Timeswap/Timeswap-V1-Core/contracts/libraries/BurnMath.sol#L41
https://github.com/code-423n4/2022-01-timeswap/blob/5960e07d39f2b4a60cfabde1bd51f4b1e62e7e85/Timeswap/Timeswap-V1-Core/contracts/libraries/WithdrawMath.sol#L33


