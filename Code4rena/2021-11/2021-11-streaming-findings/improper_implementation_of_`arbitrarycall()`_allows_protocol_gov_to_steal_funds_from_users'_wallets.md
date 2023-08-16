## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Improper implementation of `arbitraryCall()` allows protocol gov to steal funds from users' wallets](https://github.com/code-423n4/2021-11-streaming-findings/issues/258) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L733-L735

```solidity
function arbitraryCall(address who, bytes memory data) public lock externallyGoverned {
    // cannot have an active incentive for the callee
    require(incentives[who] == 0, "inc");
    ...
```

When an incentiveToken is claimed after `endStream`, `incentives[who]` will be `0` for that `incentiveToken`.

If the protocol gov is malicious or compromised, they can call `arbitraryCall()` with the address of the incentiveToken as `who` and `transferFrom()` as calldata and steal all the incentiveToken in the victim's wallet balance up to the allowance amount.

### PoC

1. Alice approved `USDC` to the streaming contract;
2. Alice called `createIncentive()` and added `1,000 USDC` of incentive;
3. After the stream is done, the stream creator called `claimIncentive()` and claimed `1,000 USDC`;

The compromised protocol gov can call `arbitraryCall()` and steal all the USDC in Alice's wallet balance.

### Recommendation

Consider adding a mapping: `isIncentiveToken`, setting `isIncentiveToken[incentiveToken] = true` in `createIncentive()`, and `require(!isIncentiveToken[who], ...)` in `arbitraryCall()`.

