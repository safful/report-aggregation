## Tags

- bug
- disagree with severity
- QA (Quality Assurance)
- sponsor confirmed

# [TRSRY.sol function repayLoan() alows only loan owner to repay loan](https://github.com/code-423n4/2022-08-olympus-findings/issues/307) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L104-L119


# Vulnerability details

### TRSRY.sol alows only loan owner to repay loan

It should be allowed that that everyone can repay the loan. There could be a situation that loan owner is not able to repay the loan but a different address could repay in his place. It seems as unnecessary restriction that only the owner can repay his loan.

**Recommendation**: Allow everyone to repay any loan.
Context: [`TRSRY.sol#L104-L119`](https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/TRSRY.sol#L104-L119)
```diff=
-    function repayLoan(ERC20 token_, uint256 amount_) external nonReentrant {
-        if (reserveDebt[token_][msg.sender] == 0) revert TRSRY_NoDebtOutstanding();

        // Deposit from caller first (to handle nonstandard token transfers)
        uint256 prevBalance = token_.balanceOf(address(this));
        token_.safeTransferFrom(msg.sender, address(this), amount_);

        uint256 received = token_.balanceOf(address(this)) - prevBalance;

        // Subtract debt from caller
-        reserveDebt[token_][msg.sender] -= received;
        totalDebt[token_] -= received;

-        emit DebtRepaid(token_, msg.sender, received);
    }
```