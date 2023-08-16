## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Validate max fee](https://github.com/code-423n4/2021-10-tracer-findings/issues/21) 

# Handle

cmichel


# Vulnerability details

`PoolFactory.setFee` does not check if the `_fee` parameter is at most 100%.

## Impact
Setting a very high fee, even above 100%, will lead to the pool's funds being drained.

## Recommended Mitigation Steps
Validate `_fee` against a reasonable max-fee value, ideally < 100%.


