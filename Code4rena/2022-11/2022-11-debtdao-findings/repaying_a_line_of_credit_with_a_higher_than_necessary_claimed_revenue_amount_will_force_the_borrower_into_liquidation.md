## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- H-06

# [Repaying a line of credit with a higher than necessary claimed revenue amount will force the borrower into liquidation](https://github.com/code-423n4/2022-11-debtdao-findings/issues/461) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/CreditLib.sol#L186


# Vulnerability details

A borrower can repay (parts) of a credit line with the `SpigotedLine.useAndRepay` function. This function will use `amount` of `unusedTokens[credit.token]` as a repayment. However, if `amount` exceeds the principal and the accrued interest, `credit.principal` will underflow without an error and set the principal value to a very large number.

This a problem because a borrower can unknowingly provide a larger than necessary `amount` to the `SpigotedLine.useAndRepay` function to make sure enough funds are used to fully repay the principal and the remaining interest.

Additionally, a lender can do the same thing as the lender can call this function.

## Impact

The `credit.principal` underflows without an error and will be set to a very large number. This will force a secured line **immediately** into liquidation. Additionally, having a principal value close to `2^256 - 1` will make it hugely expensive to repay the credit line.

## Proof of Concept

[utils/CreditLib.sol#L186](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/utils/CreditLib.sol#L186)

```solidity
function repay(
  ILineOfCredit.Credit memory credit,
  bytes32 id,
  uint256 amount
)
  external
  returns (ILineOfCredit.Credit memory)
{ unchecked {
    if (amount <= credit.interestAccrued) {
        credit.interestAccrued -= amount;
        credit.interestRepaid += amount;
        emit RepayInterest(id, amount);
        return credit;
    } else {
        uint256 interest = credit.interestAccrued;
        uint256 principalPayment = amount - interest;

        // update individual credit line denominated in token
        credit.principal -= principalPayment; // @audit-info potential underflow without an error due to the unchecked block
        credit.interestRepaid += interest;
        credit.interestAccrued = 0;

        emit RepayInterest(id, interest);
        emit RepayPrincipal(id, principalPayment);

        return credit;
    }
} }
```

To demonstrate the issue, copy the following test case and paste it into the `SpigotedLine.t.sol` test file. Then run `forge test --match-test "test_lender_use_and_repay_underflow"`.

Following scenario causes the repayment to underflow:

1. Borrower borrows `1 ether` of `revenueToken`
2. `2 ether` worth of `revenueToken` is claimed and traded from the revenue contract
3. Use all of the previously claimed funds (`2 ether`) to repay the line of credit (= `1 ether`)
4. `credit.principal` underflows due to `principalPayment` is larger than `credit.principal`

```solidity
function test_lender_use_and_repay_underflow() public {
    uint256 largeRevenueAmount = lentAmount * 2;

    deal(address(lender), lentAmount + 1 ether);
    deal(address(revenueToken), MAX_REVENUE);
    address revenueC = address(0xbeef); // need new spigot for testing
    bytes32 id = _createCredit(address(revenueToken), Denominations.ETH, revenueC);

    // 1. Borrow lentAmount = 1 ether
    _borrow(id, lentAmount);

    // 2. Claim and trade largeRevenueAmount = 2 ether (revenue)
    bytes memory tradeData = abi.encodeWithSignature(
      'trade(address,address,uint256,uint256)',
      address(revenueToken),
      Denominations.ETH,
      1 gwei,
      largeRevenueAmount
    );

    hoax(borrower);
    line.claimAndTrade(address(revenueToken), tradeData);

    (, uint256 principalBeforeRepaying,,,,,) = line.credits(line.ids(0));
    assertEq(principalBeforeRepaying, lentAmount);

    // 3. Use and repay debt with previously claimed and traded revenue (largeRevenueAmount = 2 ether)
    vm.prank(lender);
    line.useAndRepay(largeRevenueAmount);
    (, uint256 _principal,,,,,) = line.credits(line.ids(0));

    uint256 underflowedPrincipal = principalBeforeRepaying;

    unchecked {
      underflowedPrincipal -= (largeRevenueAmount);
    }

    // 4. Principal underflowed
    assertEq(_principal, underflowedPrincipal);
  }
```

## Tools Used

Manual review

## Recommended mitigation steps

Consider asserting `amount` is less or equal than `credit.principal + credit.interestAccrued` (`require(amount <= credit.principal + credit.interestAccrued);`). Similar as how it is done in [`LineOfCredit.depositAndRepay()`](https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L326)
