## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Cached version of ovl may be outdated](https://github.com/code-423n4/2021-11-overlay-findings/issues/129) 

# Handle

pauliax


# Vulnerability details

## Impact
contract OverlayV1OVLCollateral and OverlayV1Governance cache ovl address:
```solidity
 IOverlayTokenNew immutable public ovl;
```
This variable is initialized in the constructor and fetched from the mothership contract:
```solidity
  mothership = IOverlayV1Mothership(_mothership);
  ovl = IOverlayV1Mothership(_mothership).ovl();
```
ovl is declared as immutable and later contract interacts with this cached version. However, mothership contains a setter function, so the governor can point it to a new address:
```solidity
function setOVL (address _ovl) external onlyGovernor {
    ovl = _ovl;
}
```

OverlayV1OVLCollateral and OverlayV1Governance will still use this old cached value.

## Recommended Mitigation Steps
Consider if this was intended, or you want to remove this cached version and always fetch on the go (this will increase the gas costs though).

