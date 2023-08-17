## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [When an attacker lends to a loan, the attacker can trigger DoS that any lenders can not buyout it](https://github.com/code-423n4/2022-04-backed-findings/issues/89) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/main/contracts/NFTLoanFacilitator.sol#L205-L208
https://github.com/code-423n4/2022-04-backed/blob/main/contracts/NFTLoanFacilitator.sol#L215-L218


# Vulnerability details

## Impact

If an attacker (lender) lends to a loan, the attacker can always revert transactions when any lenders try to buyout, making anyone can not buyout the loan of the attacker.

## Proof of Concept

1. A victim calls `lend()`, trying to buyout the loan of the attacker.
2. In `lend()`, it always call `ERC20(loanAssetContractAddress).safeTransfer` to send `accumulatedInterest + previousLoanAmount` to `currentLoanOwner` (attacker).
3. If the `transfer` of `loanAssetContractAddress` is ERC777, it will call `_callTokensReceived` that the attacker can manipulate and always revert it.
4. Because `NFTLoanFacilitator` uses `safeTransfer` and `safeTransferFrom` to check return value, the transaction of the victim will also be reverted. It makes anyone can not buyout the loan of the attacker.

In `_callTokensReceived`, the attacker just wants to revert the buyout transaction, but keep `repayAndCloseLoan` successful. The attacker can call `loanInfoStruct(uint256 loanId)` in `_callTokensReceived` to check if the value of `loanInfo` is changed or not to decide to revert it.

## Tools Used

vim

## Recommended Mitigation Steps

Don't transfer `ERC20(loanAssetContractAddress)` to `currentLoanOwner` in `lend()`, use a global mapping to record redemption of lenders and add an external function `redeem` for lenders to transfer `ERC20(loanAssetContractAddress)`.


