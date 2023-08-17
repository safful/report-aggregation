## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed
- Notional

# [wfCash4626 withdraw method can settle the account](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/169) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/main/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L192


# Vulnerability details

`withdraw` will revert if the account has not been settled yet.
This is just due to the implementation and can be avoided by, well, settling the account.

## Impact
`withdraw` reverts unnecessarily. Protocols and users which will use wfCash4626 will have to discover this and settle by themselves.

## Proof of Concept
`withdraw` [calls](https://github.com/code-423n4/2022-06-notional-coop/blob/main/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L192) `previewWithdraw`, which ends up calling `_getMaturedValue`, which [will revert](https://github.com/code-423n4/2022-06-notional-coop/blob/main/notional-wrapped-fcash/contracts/wfCashERC4626.sol#L23) if the account is not settled yet.

## Recommended Mitigation Steps
Add to `withdraw`:
```
NotionalV2.settleAccount(address(this));
```
This will ensure that the account is settled and `withdraw` will not revert.

