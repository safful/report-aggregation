## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [denial fo service](https://github.com/code-423n4/2022-02-hubble-findings/issues/119) 

# Lines of code

https://github.com/code-423n4/2022-02-hubble/blob/main/contracts/VUSD.sol#L53


# Vulnerability details

processWithdrawals can process limited amount in each call.
an attacker can push to withdrawals enormous amount of withdrawals with amount = 0.
in order to stop the dos attack and process the withdrawal, the governance needs to spend as much gas as the attacker.
if the governance doesn't have enough money to pay for the gas, the withdrawals can't be processed.

## Proof of Concept
Alice wants to attack vusd, she spend 1millions dollars for gas to push as many withdrawals of amount = 0 as she can.
if the governance wants to process the deposits after alices empty deposits, they also need to spend at least 1 million dollars for gas in order to process alice's withdrawals first.
but the governance doesn't have 1 million dollar so the funds will be locked.

## Recommended Mitigation Steps
set  a minimum amount of withdrawal. e.g. 1 dollar

```
    function withdraw(uint amount) external {
        require(amount >= 10 ** 6);
        burn(amount);
        withdrawals.push(Withdrawal(msg.sender, amount));
    }
```

