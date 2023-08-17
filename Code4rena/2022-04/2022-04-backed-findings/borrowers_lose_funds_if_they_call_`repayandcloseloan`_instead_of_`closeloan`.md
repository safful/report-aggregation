## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Borrowers lose funds if they call `repayAndCloseLoan` instead of `closeLoan`](https://github.com/code-423n4/2022-04-backed-findings/issues/27) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/e8015d7c4b295af131f017e646ba1b99c8f608f0/contracts/NFTLoanFacilitator.sol#L241


# Vulnerability details

## Impact
The `repayAndCloseLoan` function does not revert if there has not been a lender for a loan (matched with `lend`).
Users should use `closeLoan` in this case but the contract should disallow calling `repayAndCloseLoan` because users can lose funds.

It performs a `ERC20(loan.loanAssetContractAddress).safeTransferFrom(msg.sender, lender, interest + loan.loanAmount)` call where `interest` will be a high value accumulated from timestamp 0 and the `loan.loanAmount` is the initially desired min loan amount `minLoanAmount` set in `createLoan`.
The user will lose these funds if they ever approved the contract (for example, for another loan).

## Recommended Mitigation Steps
Add a check that there actually is something to repay.

```solidity
require(loan.lastAccumulatedTimestamp > 0, "loan was never matched by a lender. use closeLoan instead");
```


