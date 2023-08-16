## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unused WJLP can't be simply unwrapped](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/138) 

# Handle

kenzo


# Vulnerability details

WJLP can only be unwrapped from the Active Pool or Stability Pool.
A user who decided to wrap his JLP, but not use all of them in a trove,
Wouldn't be able to just unwrap them.

## Impact
Impaired functionality for users.
Would have to incur fees for simple unwrapping.

## Proof of Concept
The unwrap functionality is only available from `unwrapFor` function, and that function is only callable from AP or SP. [(Code ref)](https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L148:#L149)
```
function unwrapFor(address _to, uint _amount) external override {
        _requireCallerIsAPorSP();
```

## Recommended Mitigation Steps
Allow anybody to call the function.
As it will burn the holder's WJLP, a user could only unwrap tokens that are not in use.

