## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Calling `unstake()` can cause locked funds](https://github.com/code-423n4/2022-06-infinity-findings/issues/50) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/staking/InfinityStaker.sol#L290-L325


# Vulnerability details

## Impact
Following scenario:

Alice has staked X token for 6 months that have vested. She stakes Y tokens for another three months. If she now calls `unstake(X)` to take out the tokens that have vested, the Y tokens she staked for three months will be locked up.

## Proof of Concept
First, here's a test showcasing the issue:

```js
  describe('should cause trouble', () => {
    it('should lock up funds', async function () {
      await approveERC20(signer1.address, token.address, amountStaked, signer1, infinityStaker.address);
      await infinityStaker.connect(signer1).stake(amountStaked, 2);
      await network.provider.send("evm_increaseTime", [181 * DAY]);
      await network.provider.send('evm_mine', []);
      
      // The funds we staked for 6 months have vested
      expect(await infinityStaker.getUserTotalVested(signer1.address)).to.eq(amountStaked);

      // Now we want to stake funds for three months
      await approveERC20(signer1.address, token.address, amountStaked2, signer1, infinityStaker.address);
      await infinityStaker.connect(signer1).stake(amountStaked2, 1);

      // total staked is now the funds staked for three & six months
      // total vested stays the same
      expect(await infinityStaker.getUserTotalStaked(signer1.address)).to.eq(amountStaked.add(amountStaked2));
      expect(await infinityStaker.getUserTotalVested(signer1.address)).to.eq(amountStaked);

      // we unstake the funds that are already vested.
      const userBalanceBefore = await token.balanceOf(signer1.address);
      await infinityStaker.connect(signer1).unstake(amountStaked);
      const userBalanceAfter = await token.balanceOf(signer1.address);

      expect(userBalanceAfter).to.eq(userBalanceBefore.add(amountStaked));

      expect(await infinityStaker.getUserTotalStaked(signer1.address)).to.eq(ethers.BigNumber.from(0));
      expect(await infinityStaker.getUserTotalVested(signer1.address)).to.eq(ethers.BigNumber.from(0));
    });
  });
```

The test implements the scenario I've described above. In the end, the user got back their `amountStaked` tokens with the `amountStaked2` tokens being locked up in the contract. The user has no tokens staked at the end.

The issue is in the `_updateUserStakedAmounts()` function:

```sol
    if (amount > noVesting) {
      userstakedAmounts[user][Duration.NONE].amount = 0;
      userstakedAmounts[user][Duration.NONE].timestamp = 0;
      amount = amount - noVesting;
      if (amount > vestedThreeMonths) {
        // MAIN ISSUE:
        // here `vestedThreeMonths` is 0. The current staked tokens are set to `0` and `amount` is decreased by `0`.
        // Since `vestedThreeMonths` is `0` we shouldn't decrease `userstakedAmounts` at all here.
        userstakedAmounts[user][Duration.THREE_MONTHS].amount = 0;
        userstakedAmounts[user][Duration.THREE_MONTHS].timestamp = 0;
        amount = amount - vestedThreeMonths;
        // `amount == vestedSixMonths` so we enter the else block
        if (amount > vestedSixMonths) {
          userstakedAmounts[user][Duration.SIX_MONTHS].amount = 0;
          userstakedAmounts[user][Duration.SIX_MONTHS].timestamp = 0;
          amount = amount - vestedSixMonths;
          if (amount > vestedTwelveMonths) {
            userstakedAmounts[user][Duration.TWELVE_MONTHS].amount = 0;
            userstakedAmounts[user][Duration.TWELVE_MONTHS].timestamp = 0;
          } else {
            userstakedAmounts[user][Duration.TWELVE_MONTHS].amount -= amount;
          }
        } else {
          // the staked amount is set to `0`.
          userstakedAmounts[user][Duration.SIX_MONTHS].amount -= amount;
        }
      } else {
        userstakedAmounts[user][Duration.THREE_MONTHS].amount -= amount;
      }
    } else {
      userstakedAmounts[user][Duration.NONE].amount -= amount;
    }
```


## Tools Used
none

## Recommended Mitigation Steps
Don't set `userstakedAmounts.amount` to `0` if none of its tokens are removed (`vestedAmount == 0`)

