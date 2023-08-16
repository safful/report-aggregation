## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Rewards can be claimed multiple times](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/3) 

# Handle

johnnycash


# Vulnerability details

## Impact

An attacker can claim its reward 256 * `epochDuration` seconds after the timestamp at which the promotion started. The vulnerability allows him to claim a reward several times to retrieve all the tokens associated to the promotion. 


## Analysis

`claimRewards()` claim rewards for a given promotion and epoch. In order to prevent a user from claiming a reward multiple times, the mapping [_claimedEpochs](https://github.com/pooltogether/v4-periphery/blob/ceadb25844f95f19f33cb856222e461ed8edf005/contracts/TwabRewards.sol#L32) keeps track of claimed rewards per user:

```
    /// @notice Keeps track of claimed rewards per user.
    /// @dev _claimedEpochs[promotionId][user] => claimedEpochs
    /// @dev We pack epochs claimed by a user into a uint256. So we can't store more than 255 epochs.
    mapping(uint256 => mapping(address => uint256)) internal _claimedEpochs;
```

(The comment is wrong, epochs are packed into a uint256 which allows **256** epochs to be stored).

`_epochIds` is an array of `uint256`. For each `_epochId` in this array, `claimRewards()` checks that the reward associated to this `_epochId` isn't already claimed thanks to 
`_isClaimedEpoch()`. [_isClaimedEpoch()](https://github.com/pooltogether/v4-periphery/blob/ceadb25844f95f19f33cb856222e461ed8edf005/contracts/TwabRewards.sol#L371) checks that the bit `_epochId` of `_claimedEpochs` is unset:

```
(_userClaimedEpochs >> _epochId) & uint256(1) == 1;
```

However, if `_epochId` is greater than 255, `_isClaimedEpoch()` always returns false. It allows an attacker to claim a reward several times.

[_calculateRewardAmount()](https://github.com/pooltogether/v4-periphery/blob/ceadb25844f95f19f33cb856222e461ed8edf005/contracts/TwabRewards.sol#L289) just makes use of `_epochId` to tell whether the promotion is over.


## Proof of Concept

The following test should result in a reverted transaction, however the transaction succeeds.

```
        it('should fail to claim rewards if one or more epochs have already been claimed', async () => {
            const promotionId = 1;

            const wallet2Amount = toWei('750');
            const wallet3Amount = toWei('250');

            await ticket.mint(wallet2.address, wallet2Amount);
            await ticket.mint(wallet3.address, wallet3Amount);

            await createPromotion(ticket.address);
            await increaseTime(epochDuration * 257);

            await expect(
                twabRewards.claimRewards(wallet2.address, promotionId, ['256', '256']),
            ).to.be.revertedWith('TwabRewards/rewards-already-claimed');
        });
```


## Tools Used

Text editor.


## Recommended Mitigation Steps

A possible fix could be to change the type of `_epochId` to `uint8` in:

- `_calculateRewardAmount()`
- `_updateClaimedEpoch()`
- `_isClaimedEpoch()`

and change the type of `_epochIds` to `uint8[]` in `claimRewards()`.

