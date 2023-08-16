## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Reserve does not correctly implement RingBuffer](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/26) 

# Handle

cmichel


# Vulnerability details

The `Reserve` does not correctly use ring buffers to get the oldest / newest elements if the array is full (observations larger than cardinality) in which case it should wrap around.

`getReserveAccumulatedBetween` always picks `reserveAccumulators[_cardinality - 1]`.
`_checkpoint` tries to write to `reserveAccumulators[cardinality++]` which will break once `cardinality` reaches `MAX_CARDINALITY`.

The `TwabLib` library has a correct `oldestTwab/newestTwab` implementation using the `RingBufferLib` that wraps around if needed.

## Impact
Anyone can send 1 wei to the reserve and call `checkpoint` on it until the `MAX_CARDINALITY` is reached.
Afterwards, trying to write any new checkpoints will fail as `_checkpoint` now tries to write to `cardinality=MAX_CARDINALITY+1` which is out of bounds of the `reserveAccumulators`.

The reserve is broken and cannot withdraw funds anymore.
The gas costs for such an attack are very high and would take ~7 years if writing every block, making it probably not worth fixing.

## Recommended Mitigation Steps
Correctly implement the ring buffer usage like in `TwabLib`.


