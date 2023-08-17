## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [amount requires to be updated to contract balance increase (1)](https://github.com/code-423n4/2022-05-factorydao-findings/issues/34) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/e22a562c01c533b8765229387894cc0cb9bed116/contracts/PermissionlessBasicPoolFactory.sol#L137-L149


# Vulnerability details

## Impact
Every time transferFrom or transfer function in ERC20 standard is called there is a possibility that underlying smart contract did not transfer the exact amount entered.
It is required to find out contract balance increase/decrease after the transfer.
This pattern also prevents from re-entrancy attack vector.

## Proof of Concept

## Tools Used

## Recommended Mitigation Steps
Recommended code:
```solidity
function fundPool(uint poolId) internal {
    Pool storage pool = pools[poolId];
    bool success = true;
    uint amount;
    for (uint i = 0; i < pool.rewardFunding.length; i++) {
        amount = getMaximumRewards(poolId, i);
        // transfer the tokens from pool-creator to this contract


        uint256 balanceBefore = IERC20(pool.rewardTokens[i]).balanceOf(address(this)); // remembering asset balance before the transfer
        IERC20(pool.rewardTokens[i]).safeTransferFrom(msg.sender, address(this), amount);
        uint256 newAmount = IERC20(pool.rewardTokens[i]).balanceOf(address(this)) - balanceBefore; // updating actual amount to the contract balance increase
        success = success && newAmount == amount; // making sure amounts match

        // bookkeeping to make sure pools don't share tokens
        pool.rewardFunding[i] += amount;
    }
    require(success, 'Token deposits failed');
}
```

