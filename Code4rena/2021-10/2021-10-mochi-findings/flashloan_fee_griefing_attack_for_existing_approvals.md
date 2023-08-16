## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Flashloan fee griefing attack for existing approvals](https://github.com/code-423n4/2021-10-mochi-findings/issues/124) 

# Handle

cmichel


# Vulnerability details

If a flashloan contract does not properly authenticate the `USDM` flashloan contract callbacks, anyone can perform a griefing attack which will lead to the caller losing tokens equal to the fees.

This is because the flashloan `receiver` is not authenticated and anyone can start flashloans on behalf of another contract.
They don't even need to approve the `usdm` contract as it uses internal `_burn` and `_transfer` functions instead of `burnFrom`/`transferFrom`.

#### POC

1. Call `FlashLoan.flashLoan(receiver=victim, ...)`.
2. Loan amount + fees will be burned/transferred from the `receiver` in `_loan`.

If fees are non-zero, it's possible to drain the victim's balance if their contract is implemented incorrectly without proper authentication checks.

#### Recommendation
This is an inherent issue with EIP-3156 which defines the interface with an arbitrary `receiver`.
Contracts should be aware to revert if the flashloan was not initiated by them.

To mitigate this issue one could use functions that work with explicit approvals from the victim, instead of using internal `_burn` and `_transfer` functions. This way, the victim must first have approved the tokens for transfer.


