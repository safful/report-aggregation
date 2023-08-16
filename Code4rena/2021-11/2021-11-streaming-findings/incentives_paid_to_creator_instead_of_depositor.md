## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Incentives paid to creator instead of depositor](https://github.com/code-423n4/2021-11-streaming-findings/issues/201) 

# Handle

gzeon


# Vulnerability details

## Impact
The documentation is unclear, but it make little sense that incentives are only paid to the stream creator instead of depositors. This make the incentives more like donation to the creator but not actually incentivizing the stream.

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L518

