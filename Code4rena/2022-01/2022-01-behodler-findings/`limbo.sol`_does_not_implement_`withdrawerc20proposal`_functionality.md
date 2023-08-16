## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [`Limbo.sol` Does Not Implement `WithdrawERC20Proposal` Functionality](https://github.com/code-423n4/2022-01-behodler-findings/issues/165) 

# Handle

kirk-baird


# Vulnerability details

## Impact

The proposal contract `WithdrawERC20Proposal` allows a the DAO to withdraw ERC20 tokens to a destination the the function [withdrawERC20()](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/Proposals/WithdrawERC20Proposal.sol#L35).

However, this function is not implemented in [Limbo.sol](https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/Limbo.sol) and thus the execution can never succeed.

## Recommended Mitigation Steps

Consider implementing this functionality in `Limbo.sol` or deleting the proposal.

