## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas optimization in Vault.addValueBatch()](https://github.com/code-423n4/2022-01-insure-findings/issues/89) 

# Handle

tqts


# Vulnerability details

## Impact
None

## Proof of Concept
The `for` loop at [L109-113](https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/Vault.sol#L109-L113) can be unrolled to remove the overhead of the loop itself, and avoid using an initialized-to-zero uint128 variable.

## Tools Used
Manual review

## Recommended Mitigation Steps
Replace L109-113 with:
```
uint256 _allocation = (_shares[0] * _attributions) / MAGIC_SCALE_1E6;
attributions[_beneficiaries[0]] += _allocation;
_allocations[0] = _allocation;

_allocation = (_shares[1] * _attributions) / MAGIC_SCALE_1E6;
attributions[_beneficiaries[1]] += _allocation;
_allocations[1] = _allocation;
```

