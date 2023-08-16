## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Storage variable unstreamed can be artificially inflated](https://github.com/code-423n4/2021-11-streaming-findings/issues/118) 

# Handle

harleythedog


# Vulnerability details

## Impact
The storage variable `unstreamed` keeps track of the global amount of deposit token in the contract that have not been streamed yet. This variable is a public variable, and users that read this variable likely want to use its value to determine whether or not they want to stake in the stream.

The issue here is that `unstreamed` is incremented on calls to `stake`, but it is not being decremented on calls to `withdraw`. As a result, a malicious user could simply stake, immediately withdraw their staked amount, and they will have increased `unstreamed`. They could do this repeatedly or with large amounts to intentionally inflate `unstreamed` to be as large as they want.

Other users would see this large amount and be deterred to stake in the stream, since they would get very little reward relative to the large amount of unstreamed deposit tokens that *appear* to be in the contract. This benefits the attacker as less users will want to stake in the stream, which leaves more rewards for them.

## Proof of Concept
See `stake` here: https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L417

See `withdraw` here: https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L455

Notice that `stake` increments `unstreamed` but `withdraw` does not affect `unstreamed` at all, even though `withdraw` is indeed removing unstreamed deposit tokens from the contract.

## Tools Used
Inspection

## Recommended Mitigation Steps
Add the following line to `withdraw` to fix this issue:
```
unstreamed -= amount;
```

