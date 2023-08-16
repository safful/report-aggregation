## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Self-transfer leads to wrong withdrawable repayments](https://github.com/code-423n4/2021-12-sublime-findings/issues/146) 

# Handle

cmichel


# Vulnerability details

When transferring pool tokens to oneself the `Pool._beforeTokenTransfer` overwrites the `effectiveInterestWithdrawn` of the user with a higher amount than expected.
It uses the previous balance + the transfer amount instead of just the previous balance:

```solidity
// @audit if from == to: overwrites with last _to statement => bug
lenders[_from].effectiveInterestWithdrawn = (_fromBalance.sub(_amount)).mul(_totalRepaidAmount).div(_totalSupply);
lenders[_to].effectiveInterestWithdrawn = (_toBalance.add(_amount)).mul(_totalRepaidAmount).div(_totalSupply);
```

# Impact
The bug is not in the user's favor and would lead to them being able to withdraw fewer repayments in the future.

#### POC
- user calls `Pool.transfer(from=user, to=user, amount=pool.balanceOf(user))`
- pending repayments are withdrawn first by the `_withdrawRepayment` calls. (second one does not lead to a second withdrawal as the `effectiveInterestWithdrawn` is already increased in the first call)
- `lenders[user].effectiveInterestWithdrawn` is then set using `2 * userBalance`.
- This has the effect that the user appears to have claimed twice as many repayments as their balance indicates already and they won't be able to claim anymore for a while.

## Recommended Mitigation Steps
We still recommend fixing this bug, for example, by disallowing self-transfers.


