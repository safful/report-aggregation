## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-03

# [Groupbuy: _verifyUnsuccessfulState and _verifySuccessfulState both can return true when block.timestamp == pool.terminationPeriod](https://github.com/code-423n4/2022-12-tessera-findings/issues/10) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/1e408ebc1c4fdcc72678ea7f21a94d38855ccc0b/src/modules/GroupBuy.sol#L455
https://github.com/code-423n4/2022-12-tessera/blob/1e408ebc1c4fdcc72678ea7f21a94d38855ccc0b/src/modules/GroupBuy.sol#L478


# Vulnerability details

## Impact
The functions `_verifyUnsuccessfulState` and `_verifySuccessfulState` should always have a differing behavior with regards to reversion, i.e. when one does not revert, the other should revert. In one condition, this is not true. Namely, when we have `pool.success == false` and `block.timestamp == pool.terminationPeriod`, this check within `_verifyUnsuccessfulState` is `false`:
```solidity
if (pool.success || block.timestamp > pool.terminationPeriod) revert InvalidState();
```
Similarly, this check within `_verifySuccessfulState` is also `false`:
```solidity
if (!pool.success && block.timestamp < pool.terminationPeriod) revert InvalidState();
```
Because this breaks a fundamental invariant of the contract, there are probably multiple ways to exploit it. 
One way an attacker can exploit is by calling `claim` (to get his contribution back completely), bidding again with a higher value than his previous contributions (to get his contributions back again).

## Proof Of Concept
Let's assume we are at timestamp `pool.terminationPeriod`. Attacker Charlie has performed the lowest bid with quantity 10 and price 1 ETH. He calls `claim` to get his 10 ETH back. Now, he calls `contribute` with a quantity of 10 and a price of 2 ETH. Because this bid is higher than his previous one (which was the lowest one), his `pendingBalances` is set to 10 ETH (for the deleted entries) and his `userContributions` is set to 20 ETH (for this new contribution). He can now call `claim` again to get back his 20 ETH in `userContributions`, but also the 10 ETH in `pendingBalances`. Like that, he has stolen 10 ETH (and could use this attack pattern to drain the whole contract).

## Recommended Mitigation Steps
Change `<` in `_verifySuccessfulState` to `<=`.