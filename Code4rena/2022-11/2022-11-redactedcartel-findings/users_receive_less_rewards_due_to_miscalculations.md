## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-02

# [Users Receive Less Rewards Due To Miscalculations](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/177) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L305
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L281
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L373


# Vulnerability details

## Background

The amount of rewards accrued by global and user states is computed by the following steps:

1. Calculate seconds elapsed since the last update (`block.timestamp - lastUpdate`)
2. Calculate the new rewards by multiplying seconds elapsed by the last supply (`(block.timestamp - lastUpdate) * lastSupply`)
3. Append the new rewards to the existing rewards (`rewards = rewards + (block.timestamp - lastUpdate) * lastSupply`)

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L305

```solidity
/**
    @notice Update global accrual state
    @param  globalState    GlobalState  Global state of the producer token
    @param  producerToken  ERC20        Producer token contract
*/
function _globalAccrue(GlobalState storage globalState, ERC20 producerToken)
	internal
{
    uint256 totalSupply = producerToken.totalSupply();
    uint256 lastUpdate = globalState.lastUpdate;
    uint256 lastSupply = globalState.lastSupply;

    // Calculate rewards, the product of seconds elapsed and last supply
    // Only calculate and update states when needed
    if (block.timestamp != lastUpdate || totalSupply != lastSupply) {
        uint256 rewards = globalState.rewards +
            (block.timestamp - lastUpdate) *
            lastSupply;
            
            globalState.lastUpdate = block.timestamp.safeCastTo32();
            globalState.lastSupply = totalSupply.safeCastTo224();
            globalState.rewards = rewards;
   	..SNIP..
}
```

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L281

```solidity
/**
    @notice Update user rewards accrual state
    @param  producerToken  ERC20    Rewards-producing token
    @param  user           address  User address
*/
function userAccrue(ERC20 producerToken, address user) public {
    if (address(producerToken) == address(0)) revert ZeroAddress();
    if (user == address(0)) revert ZeroAddress();

    UserState storage u = producerTokens[producerToken].userStates[user];
    uint256 balance = producerToken.balanceOf(user);

    // Calculate the amount of rewards accrued by the user up to this call
    uint256 rewards = u.rewards +
    u.lastBalance *
    (block.timestamp - u.lastUpdate);
    
    u.lastUpdate = block.timestamp.safeCastTo32();
    u.lastBalance = balance.safeCastTo224();
    u.rewards = rewards;
    ..SNIP..
}
```

When a user claims the rewards, the number of reward tokens the user is entitled to is equal to the `rewardState` scaled by the ratio of the `userRewards` to the `globalRewards`. Refer to Line 403 below.

The `rewardState` represents the total number of a specific ERC20 reward token (e.g. WETH or esGMX) held by a producer (e.g. pxGMX or pxGPL). 

The `rewardState` of each reward token (e.g. WETH or esGMX) will increase whenever the rewards are harvested by the producer (e.g. `PirexRewards.harvest` is called). On the other hand, the `rewardState` will decrease if the users claim the rewards.

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/PirexRewards.sol#L373

```solidity
File: PirexRewards.sol
373:     function claim(ERC20 producerToken, address user) external {
..SNIP..
395:             // Transfer the proportionate reward token amounts to the recipient
396:             for (uint256 i; i < rLen; ++i) {
397:                 ERC20 rewardToken = rewardTokens[i];
398:                 address rewardRecipient = p.rewardRecipients[user][rewardToken];
399:                 address recipient = rewardRecipient != address(0)
400:                     ? rewardRecipient
401:                     : user;
402:                 uint256 rewardState = p.rewardStates[rewardToken];
403:                 uint256 amount = (rewardState * userRewards) / globalRewards;
..SNIP..
417:     }
```

#### How reward tokens are distributed

The Multiplier Point (MP) effect will be ignored for simplicity. Assume that the emission rate is constant throughout the entire period (from T80 to T84) and the emission rate is 1 esGMX per 1 GMX staked per second.

The graph below represents the amount of GMX tokens Alice and Bob staked for each second during the period. 

A = Alice and B = Bob; each block represents 1 GMX token staked.

![](https://user-images.githubusercontent.com/102820284/204132445-50422095-c02c-4f45-95d6-575667211092.png)

Based on the above graph:

- Alice staked 1 GMX token from T80 to T84. Alice will earn five (5) esGMX tokens at the end of T84.
- Bob staked 4 GMX tokens from T83 to T84. Bob will earn eight (8) esGMX tokens at the end of T84.
- A total of 13 esGMX will be harvested by `PirexRewards` contract at the end of T84

The existing reward distribution design in the `PirexRewards` contract will work perfectly if the emission rate is constant, similar to the example above.

In this case, the state variable will be as follows at the end of T84, assuming both the global and all user states have been updated and rewards have been harvested.

- rewardState = 13 esGMX tokens (5 + 8)
- globalRewards = 13
- Accrued `userRewards` of Alice = 5
- Accrued `userRewards` of Bob = 8

When Alice calls the `PirexRewards.claim` function to claim her rewards at the end of T84, she will get back five (5) esGMX tokens, which is correct.

```solidity
(rewardState * userRewards) / globalRewards
(13 * 5) / 13 = 5
```

## Proof of Concept

However, the fact is that the emission rate of reward tokens (e.g. esGMX or WETH) is not constant. Instead, the emission rate is dynamic and depends on various factors, such as the following:

- The number of rewards tokens allocated by GMX governance for each month. Refer to https://gov.gmx.io/t/esgmx-emissions/272. In some months, the number of esGMX emissions will be higher.
- The number of GMX/GLP tokens staked by the community. The more tokens being staked by the community users, the more diluted the rewards will be.

The graph below represents the amount of GMX tokens Alice and Bob staked for each second during the period. 

A = Alice and B = Bob; each block represents 1 GMX token staked.

![](https://user-images.githubusercontent.com/102820284/204132445-50422095-c02c-4f45-95d6-575667211092.png)

The Multiplier Point (MP) effect will be ignored for simplicity. Assume that the emission rate is as follows:

- From T80 to 82: 2 esGMX per 1 GMX staked per second (Higher emission rate)
- From T83 to 84: 1 esGMX per 1 GMX staked per second (Lower emission rate)

By manually computing the amount of esGMX reward tokens that Alice is entitled to at the end of T84:

```solidity
[1 staked GMX * (T82 - T80) * 2esGMX/sec] + [1 staked GMX * (T84 - T83) * 1esGMX/sec]
[1 staked GMX * 3 secs * 2esGMX/sec] + [1 staked GMX * 2secs * 1esGMX/sec]
6 + 2 = 8
```

Alice will be entitled to 8 esGMX reward tokens at the end of T84.

By manually computing the amount of esGMX reward tokens that Bob is entitled to at the end of T84:

```solidity
[4 staked GMX * 2secs * 1esGMX/sec] = 8
```

Bob will be entitled to 8 esGMX reward tokens at the end of T84.

However, the existing reward distribution design in the `PirexRewards` contract will cause Alice to get fewer reward tokens than she is entitled to and cause Bob to get more rewards than he is entitled to.

The state variable will be as follows at the end of T84, assuming both the global and all user states have been updated and rewards have been harvested.

- rewardState = 16 esGMX tokens (8 + 8)
- globalRewards = 13
- Accrued `userRewards` of Alice = 5
- Accrued `userRewards` of Bob = 8

When Alice calls the `PirexRewards.claim` function to claim her rewards at the end of T84, she will only get back six (6) esGMX tokens, which is less than eight (8) esGMX tokens she is entitled to or earned.

```solidity
(rewardState * userRewards) / globalRewards
(16 * 5) / 13 = 6.15 = 6
```

When Bob calls the `PirexRewards.claim` function to claim his rewards at the end of T84, he will get back nine (9) esGMX tokens, which is more than eight (8) esGMX tokens he is entitled to or earned.

```solidity
(rewardState * userRewards) / globalRewards
(16 * 8) / 13 = 9.85 = 9
```

## Impact

As shown in the PoC, some users will lose their reward tokens due to the miscalculation within the existing reward distribution design.

## Recommended Mitigation Steps

Update the existing reward distribution design to handle the dynamic emission rate. Implement the RewardPerToken for users and global, as seen in many of the well-established reward contracts below, which is not vulnerable to this issue:

- https://github.com/fei-protocol/flywheel-v2/blob/dbe3cb81a3dc2e46536bb8af9c2bdc585f63425e/src/FlywheelCore.sol#L226
- https://github.com/Synthetixio/synthetix/blob/2cb4b23fe409af526de67dfbb84aae84b2b13747/contracts/StakingRewards.sol#L61