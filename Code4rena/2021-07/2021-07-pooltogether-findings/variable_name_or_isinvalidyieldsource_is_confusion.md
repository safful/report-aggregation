## Tags

- bug
- 1 (Low Risk)
- SwappableYieldSource
- sponsor confirmed

# [Variable name or isInvalidYieldSource is confusion](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _requireYieldSource of the contract SwappableYieldSource has a state variable: isInvalidYieldSource

You would expect isInvalidYieldSource == true would mean the yield source in invalid
However in the source code  isInvalidYieldSource == true mean the yield source is valid.

This is confusing for readers and future maintainers. Future maintainers could easily make a mistake and thus introduce vulnerabilities.

## Proof of Concept
// https://github.com/pooltogether/swappable-yield-source/blob/main/contracts/SwappableYieldSource.sol#L74
function _requireYieldSource(IYieldSource _yieldSource) internal view {
    require(address(_yieldSource) != address(0), "SwappableYieldSource/yieldSource-not-zero-address");
    (, bytes memory depositTokenAddressData) = address(_yieldSource).staticcall(abi.encode(_yieldSource.depositToken.selector));
    bool isInvalidYieldSource;
    if (depositTokenAddressData.length > 0) {
      (address depositTokenAddress) = abi.decode(depositTokenAddressData, (address));
      isInvalidYieldSource = depositTokenAddress != address(0);
    }
    require(isInvalidYieldSource, "SwappableYieldSource/invalid-yield-source");
  }

## Tools Used

## Recommended Mitigation Steps
Change isInvalidYieldSource to isValidYieldSource 

