## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Gas: `ExchangeFactory.feeAddress()` should be declared external](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/26) 

# Handle

Dravee


# Vulnerability details

## Impact
Public functions that are never called by the contract should be declared external to save gas.

## Proof of Concept
Instances include:
```
File: ExchangeFactory.sol
81:     function feeAddress() public view virtual override returns (address) {
82:         return feeAddress_;
83:     }
```

## Tools Used
Slither

## Recommended Mitigation Steps
Change the visibility to `external`

