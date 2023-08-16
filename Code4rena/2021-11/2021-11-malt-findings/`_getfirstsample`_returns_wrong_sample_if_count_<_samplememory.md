## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [`_getFirstSample` returns wrong sample if count < sampleMemory](https://github.com/code-423n4/2021-11-malt-findings/issues/252) 

# Handle

cmichel


# Vulnerability details

The `MovingAverage.sol` contract defines several variables that in the end make the `samples` array act as a ring buffer:
- `sampleMemory`: The total length (buffer size) of the `samples` array. `samples` is initialized with `sampleMemory` zero observations.
- `counter`: The pending sample index (modulo `sampleMemory`)

The `_getFirstSample` function computes the first sample as `(counter + 1) % sampleMemory` which returns the correct index only _if the ring buffer is full_, i.e., it wraps around. (in the `counter + 1 >= sampleMemory`).

If the `samples` array does not wrap around yet, the zero index should be returned instead.

## Impact
Returning `counter + 1` if `counter + 1 < sampleMemory` returns a zero initialized `samples` observation index.
This then leads to a wrong computation of the TWAP.

## Recommended Mitigation Steps
Add an additional check for `if (counter + 1 < sampleMemory) return 0` in `_getFirstSample`.

