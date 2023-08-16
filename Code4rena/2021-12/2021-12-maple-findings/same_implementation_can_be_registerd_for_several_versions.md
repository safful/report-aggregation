## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Same implementation can be registerd for several versions](https://github.com/code-423n4/2021-12-maple-findings/issues/33) 

# Handle

cmichel


# Vulnerability details

It's possible to overwrite the `_versionOf[implementationAddress_]` field through the `proxy-factory-1.0.0-beta.1/contracts/ProxyFactory._registerImplementation` function and register the implementation as several distinct versions.

```solidity
function _registerImplementation(uint256 version_, address implementationAddress_) internal virtual returns (bool success_) {
    // Cannot already be registered and cannot be empty implementation.
    if (_implementationOf[version_] != address(0) || !_isContract(implementationAddress_)) return false;
    _versionOf[implementationAddress_] = version_;
    _implementationOf[version_]        = implementationAddress_;

    return true;
}
```

#### POC
- call `_registerImplementation(1, impl)`
- call `_registerImplementation(2, impl)`. This does not check that the versions has not already been registered by checking `_versionOf[impl] == 0`. Then the old `_versionOf[impl] = 1` is overwritten with `2`.

## Recommended Mitigation Steps
Check if being able to set a new version for the same contract is desired. If not, add a `_versionOf[impl] == 0` check.

