## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Check on `poolConstants.loanWithdrawalDeadline` for liquidation is unnecessary](https://github.com/code-423n4/2021-12-sublime-findings/issues/66) 

# Handle

TomFrenchBlockchain


# Vulnerability details

## Impact

Gas costs

## Proof of Concept

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L322-L340

In `Pool.withdrawBorrowedAmount` we set the loan to active and delete the withdrawal deadline (see link).

https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/Pool/Pool.sol#L802

When checking whether we can liquidate the pool we check that the loan is active and that `block.timestamp > poolConstants.loanWithdrawalDeadline`. This second condition always resolves true as `poolConstants.loanWithdrawalDeadline = 0` after deletion. We can then save an SLOAD by skipping this second check.

## Recommended Mitigation Steps

As above.

