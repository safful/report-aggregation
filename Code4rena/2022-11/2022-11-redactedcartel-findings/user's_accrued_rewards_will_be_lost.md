## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-04

# [User's Accrued Rewards Will Be Lost](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/184) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L373


# Vulnerability details

## Proof of Concept

If the user deposits too little GMX compared to other users (or total supply of pxGMX), the user will not be able to receive rewards after calling the `PirexRewards.claim` function. Subsequently, their accrued rewards will be cleared out (set to zero), and they will lose their rewards.

The amount of reward tokens that are claimable by a user is computed in Line 403 of the `PirexRewards.claim` function.

If the balance of pxGMX of a user is too small compared to other users (or total supply of pxGMX), the code below will always return zero due to rounding issues within solidity.

```solidity
uint256 amount = (rewardState * userRewards) / globalRewards;
```

Since the user's accrued rewards is cleared at Line 391 within the `PirexRewards.claim` function (`p.userStates[user].rewards = 0;`), the user's accrued rewards will be lost.

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L373

```solidity
File: PirexRewards.sol
368:     /**
369:         @notice Claim rewards
370:         @param  producerToken  ERC20    Producer token contract
371:         @param  user           address  User
372:     */
373:     function claim(ERC20 producerToken, address user) external {
374:         if (address(producerToken) == address(0)) revert ZeroAddress();
375:         if (user == address(0)) revert ZeroAddress();
376: 
377:         harvest();
378:         userAccrue(producerToken, user);
379: 
380:         ProducerToken storage p = producerTokens[producerToken];
381:         uint256 globalRewards = p.globalState.rewards;
382:         uint256 userRewards = p.userStates[user].rewards;
383: 
384:         // Claim should be skipped and not reverted on zero global/user reward
385:         if (globalRewards != 0 && userRewards != 0) {
386:             ERC20[] memory rewardTokens = p.rewardTokens;
387:             uint256 rLen = rewardTokens.length;
388: 
389:             // Update global and user reward states to reflect the claim
390:             p.globalState.rewards = globalRewards - userRewards;
391:             p.userStates[user].rewards = 0;
392: 
393:             emit Claim(producerToken, user);
394: 
395:             // Transfer the proportionate reward token amounts to the recipient
396:             for (uint256 i; i < rLen; ++i) {
397:                 ERC20 rewardToken = rewardTokens[i];
398:                 address rewardRecipient = p.rewardRecipients[user][rewardToken];
399:                 address recipient = rewardRecipient != address(0)
400:                     ? rewardRecipient
401:                     : user;
402:                 uint256 rewardState = p.rewardStates[rewardToken];
403:                 uint256 amount = (rewardState * userRewards) / globalRewards;
404: 
405:                 if (amount != 0) {
406:                     // Update reward state (i.e. amount) to reflect reward tokens transferred out
407:                     p.rewardStates[rewardToken] = rewardState - amount;
408: 
409:                     producer.claimUserReward(
410:                         address(rewardToken),
411:                         amount,
412:                         recipient
413:                     );
414:                 }
415:             }
416:         }
417:     }
```

The graph below represents the amount of GMX tokens Alice and Bob staked in `PirexGmx` for each second during the period. Note that the graph is not drawn proportionally.

Green = Number of GMX tokens staked by Alice

Blue = Number of GMX tokens staked by Bob

![](https://user-images.githubusercontent.com/102820284/204132852-f76c8959-5040-46bf-9529-edd0d4a98e41.png)

Based on the above graph:

- Alice staked 1 GMX token for 4 seconds (From T80 to T85)
- Bob staked 99999 GMX tokens for 4 seconds (From T80 to T85)

Assume that the emission rate is 0.1 esGMX per 1 GMX staked per second.

In this case, the state variable will be as follows at the end of T83, assuming both the global and all user states have been updated and rewards have been harvested.

- rewardState = 60,000 esGMX tokens (600,000 * 0.1)
- globalRewards = 600,000 (100,000 * 6)
- Accrued `userRewards` of Alice = 6
- Accrued `userRewards` of Bob = 599,994 (99,999 * 6)

Following is the description of `rewardState` for reference:

> The `rewardState` represents the total number of a specific ERC20 reward token (e.g. WETH or esGMX) held by a producer (e.g. pxGMX or pxGPL). 
>
> The `rewardState` of each reward token (e.g. WETH or esGMX) will increase whenever the rewards are harvested by the producer (e.g. `PirexRewards.harvest` is called). On the other hand, the `rewardState` will decrease if the users claim the rewards.

At the end of T85, Alice should be entitled to 1.2 esGMX tokens (0.2/sec * 6).

Following is the formula used in the `PirexRewards` contract to compute the number of reward tokens a user is entitled to. 

```solidity
amount = (rewardState * userRewards) / globalRewards;
```

If Alice claims the rewards at the end of T85, she will get zero esGMX tokens instead of 1.2 esGMX tokens.

```solidity
amount = (rewardState * userRewards) / globalRewards;
60,000 * 6 / 600,000
360,000/600,000 = 0.6 = 0
```

Since Alice's accrued rewards are cleared at Line 391 within the `PirexRewards.claim` function (`p.userStates[user].rewards = 0;`), Alice's accrued rewards will be lost. Alice will have to start accruing the rewards from zero after calling the `PirexRewards.claim` function.

Another side effect is that since the 1.2 esGMX tokens that belong to Alice are still in the contract, they will be claimed by other users.

## Impact

Users who deposit too little GMX compared to other users (or total supply of pxGMX), the user will not be able to receive rewards after calling the `PirexRewards.claim` function. Also, their accrued rewards will be cleared out (set to zero). Loss of reward tokens for the users.

Additionally, the `PirexRewards.claim` function is permissionless, and anyone can trigger the claim on behalf of any user. A malicious user could call the `PirexRewards.claim` function on behalf of a victim at the right time when the victim's accrued reward is small enough to cause a rounding error or precision loss, thus causing the victim accrued reward to be cleared out (set to zero).

## Recommended Mitigation Steps

Following are some of the possible remediation actions:

#### 1. Use `RewardPerToken ` approach

Avoid calculating the rewards that the users are entitled based on the ratio of `userRewards` and `globalRewards`.

Instead, consider implementing the RewardPerToken for users and global, as seen in many of the well-established reward contracts below, which is not vulnerable to this issue:

- https://github.com/fei-protocol/flywheel-v2/blob/dbe3cb81a3dc2e46536bb8af9c2bdc585f63425e/src/FlywheelCore.sol#L226
- https://github.com/Synthetixio/synthetix/blob/2cb4b23fe409af526de67dfbb84aae84b2b13747/contracts/StakingRewards.sol#L61

#### 2. Fallback logic if`amount ==  0`

If the `amount` is zero, revert the transaction. Alternatively, if the `amount` is zero, do not clear out the user's accrued reward state variable since the user did not receive anything yet.

```diff
function claim(ERC20 producerToken, address user) external {
..SNIP..
			uint256 amount = (rewardState * userRewards) / globalRewards;

			if (amount != 0) {
				// Update reward state (i.e. amount) to reflect reward tokens transferred out
				p.rewardStates[rewardToken] = rewardState - amount;

				producer.claimUserReward(
					address(rewardToken),
					amount,
					recipient
				);
-			}
+			} else {
+				revert ZeroRewardTokens();
+			}
..SNIP..
}
```