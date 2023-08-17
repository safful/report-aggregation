## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [accountant address can be set to zero by anyone leading to loss of funds/tokens](https://github.com/code-423n4/2022-06-canto-findings/issues/117) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L14-L21
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L31
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L96
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L178
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/CNote.sol#L258


# Vulnerability details

## Impact
In CNote._setAccountantContract() , the require() check only works when `address(_accountant) != address(0)` , leading to the ability to set `_accountant` state variable to the zero address, as well as setting admin to zero address.

The following below are impacts arising from above:
## A. Users can gain underlying asset tokens for free by minting CToken in `mintFresh()` then calling `redeemFresh()`

## Proof of Concept
1. Alice calls `_setAccountantContract()` with parameter input as 0.
2. The _accountant state variable is now 0.
3. Alice/or a contract calls `mintFresh()` with input address 0 and mintAmount 1000. (assuming function is external, reporting a separate issue on the mutability)
4. This passes the `if (minter == address(_accountant))` and proceeds to mint 1000 CTokens to address(0)
5. Alice then calls `redeemFresh()` with her address as the `redeemer` parameter, and redeemTokensIn as 1000.
6. Assume exchangeRate is 1, Alice would receive 1000 tokens in underlying asset.



## B. Users could borrow CToken asset for free
A user can borrow CToken asset from the contract, then set _accountant to 0 after. With _accountant being set to 0 , the borrower , then call `repayBorrowFresh()` to have _accountant (address 0) to repay back the borrowed tokens assuming address(0) already has some tokens, and user's borrowed asset (all/part) are repaid.

## Proof of Concept
1. Alice calls `borrowFresh()` to borrow 500 CTokens from contract.
2. Then Alice calls `_setAccountantContract()` with parameter input as 0.
2. The _accountant state variable is now 0.
3. With _accountant being set to 0, Alice calls `repayBorrowFresh()` having the payer be address 0, borrower being her address and 500 as repayAmount.
4. Assume address 0 already holds 1000 CTokens, Alice's debt will be fully repaid and she'll gain 500 CTokens for free.


## C. Accounting contract could loses funds/tokens
When the _accountant is set to 0, CTokens/CNote will be sent to the zero address making the Accounting contract lose funds whenever  `doTransferOut` is called.



## Tools Used
Manual review

## Recommended Mitigation Steps
Instead of a `if (address(_accountant) != address(0))` statement, an additional require check to ensure `accountant_` parameter is not 0 address can be used in addition to the require check for caller is admin.

Change this 
```if (address(_accountant) != address(0)){
            require(msg.sender == admin, "CNote::_setAccountantContract:Only admin may call this function");
        }
```

to this
```
require(msg.sender == admin, "CNote::_setAccountantContract:Only admin may call this function");
require(accountant_ != address(0), "accoutant can't be zero address");
```

