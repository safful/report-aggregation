## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Uninitialized `RocketJoeStaking.lastRewardTimestamp` can inflate `rJoe` supply](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/202) 

# Handle

cmichel


# Vulnerability details

The `RocketJoeStaking.lastRewardTimestamp` is initialized to zero. Usually, this does not matter as `updatePool` is called before the first deposit and when `joeSupply = joe.balanceOf(address(this)) == 0`, it is set to the current time.

```solidity
function updatePool() public {
    if (block.timestamp <= lastRewardTimestamp) {
        return;
    }
    uint256 joeSupply = joe.balanceOf(address(this));

    // @audit lastRewardTimestamp is not initialized. can send 1 Joe to this contract directly => lots of rJoe minted to this contract
    if (joeSupply == 0) {
        lastRewardTimestamp = block.timestamp;
        return;
    }
    uint256 multiplier = block.timestamp - lastRewardTimestamp;
    uint256 rJoeReward = multiplier * rJoePerSec;
    accRJoePerShare =
        accRJoePerShare +
        (rJoeReward * PRECISION) /
        joeSupply;
    lastRewardTimestamp = block.timestamp;

    rJoe.mint(address(this), rJoeReward);
}
```

However, if a user first directly transfers `Joe` tokens to the contract before the first `updatePool` call, the `block.timestamp - lastRewardTimestamp = block.timestamp` will be a large timestamp value and lots of `rJoe` will be minted (but not distributed to users).
Even though they are not distributed to the users, inflating the `rJoe` total supply might not be desired.

#### Recommendation
Consider tracking the actual total deposits in a storage variable and using this value instead of the current balance for `joeSupply`.
This way, transferring tokens to the contract has no influence and depositing through `deposit` first calls `updatePool` and initializes `lastRewardTimestamp`.


