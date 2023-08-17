## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [ConvexMasterChef's deposit and withdraw can be reentered drawing all reward funds from the contract if reward token allows for transfer flow control](https://github.com/code-423n4/2022-05-aura-findings/issues/313) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L209-L221
https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L239-L250


# Vulnerability details

Reward token accounting update in deposit() and withdraw() happens after reward transfer. If reward token allows for the control of transfer call flow or can be upgraded to allow it in the future (i.e. have or can introduce the _beforetokentransfer, _afterTokenTransfer type of hooks; or, say, can be upgraded to ERC777), the current implementation makes it possible to drain all the reward token funds of the contract by directly reentering deposit() or withdraw() with tiny _amount.

Setting the severity to medium as this is conditional to transfer flow control assumption, but the impact is the full loss of contract reward token holdings.

## Proof of Concept

Both withdraw() and deposit() have the issue, performing late accounting update and not controlling for reentrancy:

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L209-L221

```solidity
    function deposit(uint256 _pid, uint256 _amount) public {
        PoolInfo storage pool = poolInfo[_pid];
        UserInfo storage user = userInfo[_pid][msg.sender];
        updatePool(_pid);
        if (user.amount > 0) {
            uint256 pending = user
                .amount
                .mul(pool.accCvxPerShare)
                .div(1e12)
                .sub(user.rewardDebt);
            safeRewardTransfer(msg.sender, pending);
        }
        pool.lpToken.safeTransferFrom(
```

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L239-L250

```solidity
    function withdraw(uint256 _pid, uint256 _amount) public {
        PoolInfo storage pool = poolInfo[_pid];
        UserInfo storage user = userInfo[_pid][msg.sender];
        require(user.amount >= _amount, "withdraw: not good");
        updatePool(_pid);
        uint256 pending = user.amount.mul(pool.accCvxPerShare).div(1e12).sub(
            user.rewardDebt
        );
        safeRewardTransfer(msg.sender, pending);
        user.amount = user.amount.sub(_amount);
        user.rewardDebt = user.amount.mul(pool.accCvxPerShare).div(1e12);
        pool.lpToken.safeTransfer(address(msg.sender), _amount);
```

## Recommended Mitigation Steps

Consider adding a direct reentrancy control, e.g. nonReentrant modifier:

https://docs.openzeppelin.com/contracts/2.x/api/utils#ReentrancyGuard

Also, consider finishing all internal state updates prior to external calls:

https://consensys.github.io/smart-contract-best-practices/attacks/reentrancy/#pitfalls-in-reentrancy-solutions

