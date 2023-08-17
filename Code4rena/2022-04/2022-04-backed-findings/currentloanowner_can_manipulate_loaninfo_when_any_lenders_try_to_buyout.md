## Tags

- bug
- help wanted
- 3 (High Risk)
- resolved
- sponsor confirmed

# [currentLoanOwner can manipulate loanInfo when any lenders try to buyout](https://github.com/code-423n4/2022-04-backed-findings/issues/88) 

# Lines of code

https://github.com/code-423n4/2022-04-backed/blob/main/contracts/NFTLoanFacilitator.sol#L205-L208
https://github.com/code-423n4/2022-04-backed/blob/main/contracts/NFTLoanFacilitator.sol#L215-L218


# Vulnerability details

## Impact

If an attacker already calls `lend()` to lend to a loan, the attacker can manipulate `loanInfo` by reentrancy attack when any lenders try to buyout. The attacker can set bad values of `lendInfo` (e.g. very long duration, and 0 interest rate) that the lender who wants to buyout don't expect.

## Proof of Concept

An attacker lends a loan, and `loanAssetContractAddress` in `loanInfo` is ERC777 which is suffering from reentrancy attack. When a lender (victim) try to buyout the loan of the attacker:

1. The victim called `lend()`.
2. In `lend()`, it always call `ERC20(loanAssetContractAddress).safeTransfer` to send `accumulatedInterest + previousLoanAmount` to `currentLoanOwner` (attacker).
3. The `transfer` of `loanAssetContractAddress` ERC777 will call `_callTokensReceived` so that the attacker can call `lend()` again in reentrancy with parameters:
    * loanId: same loan Id
    * interestRate: set to bad value (e.g. 0)
    * amount: same amount
    * durationSeconds: set to bad value (e.g. a long durationSeconds)
    * sendLendTicketTo: same address of the attacker (`currentLoanOwner`)
4. Now the variables in `loanInfo` are changed to bad value, and the victim will get the lend ticket but the loan term is manipulated, and can not set it back (because it requires a better term).

## Tools Used

vim

## Recommended Mitigation Steps

Use `nonReentrant` modifier on `lend()` to prevent reentrancy attack:

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol


