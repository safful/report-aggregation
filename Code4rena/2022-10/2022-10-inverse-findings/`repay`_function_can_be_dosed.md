## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-05

# [`repay` function can be DOSed](https://github.com/code-423n4/2022-10-inverse-findings/issues/252) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/main/src/Market.sol#L531


# Vulnerability details

## Impact
In `repay()` users can repay their debt.
```
function repay(address user, uint amount) public {
        uint debt = debts[user];
        require(debt >= amount, "Insufficient debt");
        debts[user] -= amount;
        totalDebt -= amount;
        dbr.onRepay(user, amount);
        dola.transferFrom(msg.sender, address(this), amount);
        emit Repay(user, msg.sender, amount);
    }
```

There is a `require` condition, that checks if the amount provided, is greater than the debt of the user. If it is, then the function reverts. This is where the vulnerability arises.

`repay` function can be frontrun by an attacker. Say an attacker pay a small amount of debt for the victim user, by frontrunning his repay transaction. Now when the victim's transaction gets executed, the `require` condition will fail, as the amount of debt is less than the amount of DOLA provided. Hence the attacker can repeat the process to DOS the victim from calling the repay function.


## Proof of Concept

1. Victim calls repay() function to pay his debt of 500 DOLA , by providing the amount as 500
2. Now attacker saw this transaction on mempool
3. Attacker frontruns the transaction, by calling repay() with amount provided as 1 DOLA
4. Attacker's transaction get's executed first due to frontrunning, which reduces the debt of the victim user to 499 DOLA
5. Now when the victim's transaction get's executed, the debt of victim has reduced to 499 DOLA, and the amount to repay provided was 500 DOLA. Now as debt is less than the amount provided, so the require function will fail, and the victim's transaction will revert.
This will prevent the victim from calling repay function

Hence an attacker can DOS the repay function for the victim user

## Tools Used
Manual review

## Recommended Mitigation Steps
Implement DOS protection