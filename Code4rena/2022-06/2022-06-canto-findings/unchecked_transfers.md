## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [Unchecked transfers](https://github.com/code-423n4/2022-06-canto-findings/issues/126) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Comptroller.sol#L1380
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegate.sol#L87
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegate.sol#L89
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L52
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L56
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CErc20.sol#L128
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CEther.sol#L150
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L52
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L56


# Vulnerability details

## Impact
Multiple calls to transfer are frequently done without checking the results. For certain ERC20 tokens, if insufficient tokens are present, no revert occurs but a result of “false” is returned. It’s important to check this, users or admin could gain or lose tokens if return value of transfer() is not checked.

The following functions are affected:
Comptroller.grantCompInternal() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Comptroller.sol#L1380
AccountantDelegate.sweepInterest() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegate.sol#L87
AccountantDelegate.sweepInterest() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegate.sol#L89
TreasuryDelegate.sendFund() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L52
TreasuryDelegate.sendFund() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L56
CErc20.sweepToken() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CErc20.sol#L128
CEther.doTransferOut() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CEther.sol#L150
TreasuryDelegate.sendFund() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L52
TreasuryDelegate.sendFund() - https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegate.sol#L56

## Tools Used
Slither and manual review

## Recommended Mitigation Steps
Check the returned values of transfers or use a SafeERC20 transfer.

