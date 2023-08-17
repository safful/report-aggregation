## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [`RewardThrottle._sendToDistributor()` reverts if one distributor is inactive.](https://github.com/code-423n4/2023-02-malt-findings/issues/29) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/RewardThrottle.sol#L602
https://github.com/code-423n4/2023-02-malt/blob/main/contracts/RewardSystem/LinearDistributor.sol#L101


# Vulnerability details

## Impact
`RewardThrottle._sendToDistributor()` reverts if one distributor is inactive.

## Proof of Concept
`RewardThrottle._sendToDistributor()` distributes the rewards to several distributors according to their allocation ratios.

```solidity
File: 2023-02-malt\contracts\RewardSystem\RewardThrottle.sol
575:   function _sendToDistributor(uint256 amount, uint256 epoch) internal {
576:     if (amount == 0) {
577:       return;
578:     }
579: 
580:     (
581:       uint256[] memory poolIds,
582:       uint256[] memory allocations,
583:       address[] memory distributors
584:     ) = bonding.poolAllocations();
585: 
586:     uint256 length = poolIds.length;ratio
587:     uint256 balance = collateralToken.balanceOf(address(this));
588:     uint256 rewarded;
589: 
590:     for (uint256 i; i < length; ++i) {
591:       uint256 share = (amount * allocations[i]) / 1e18;
592: 
593:       if (share == 0) {
594:         continue;
595:       }
596: 
597:       if (share > balance) {
598:         share = balance;
599:       }
600: 
601:       collateralToken.safeTransfer(distributors[i], share);
602:       IDistributor(distributors[i]).declareReward(share); //@audit will revert if one distributor is inactive
```

And `LinearDistributor.declareReward()` has an `onlyActive` modifier and it will revert in case of `inactive`.

```solidity
File: 2023-02-malt\contracts\RewardSystem\LinearDistributor.sol
098:   function declareReward(uint256 amount)
099:     external
100:     onlyRoleMalt(REWARDER_ROLE, "Only rewarder role")
101:     onlyActive
102:   {
```

As a result, `RewardThrottle._sendToDistributor()` will revert if one distributor is inactive rather than working with active distributors only.

## Tools Used
Manual Review

## Recommended Mitigation Steps
I think it's logical to continue to work with active distributors in `_sendToDistributor()`.