## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Incorrect multiplication in `_computeSingleOutGivenPoolIn` of `IndexPool`](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/166) 

# Handle

broccoli


# Vulnerability details

## Impact

The `_computeSingleOutGivenPoolIn` function of `IndexPool` uses the raw multiplication (i.e., `*`) to calculate the `zaz` variable. However, since both `(BASE - normalizedWeight)` and `_swapFee` are in `WAD`, the `_mul` function should be used instead to calculate the correct value of `zaz`. Otherwise, `zaz` would be `10 ** 18` times larger than the expected value and causes an integer underflow when calculating `amountOut`. The incorrect usage of multiplication prevents anyone from calling the function successfully.

## Proof of Concept

Referenced code:
[IndexPool.sol#L282](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/IndexPool.sol#L282)


## Recommended Mitigation Steps

Change `(BASE - normalizedWeight) * _swapFee` to `_mul((BASE - normalizedWeight), _swapFee)`.

