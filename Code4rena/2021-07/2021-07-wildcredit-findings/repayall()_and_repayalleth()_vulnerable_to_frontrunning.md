## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [repayAll() and repayAllETH() vulnerable to frontrunning](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/125) 

# Handle

toastedsteaksandwich


# Vulnerability details

## Impact
The repayAll() and repayAllETH() functions allow any user to pay off debt of another user. Since all of the debt is going to be paid, no amount is specified, allowing the recipient of the repayment to frontrun the transaction to increase their debt. The risk of this issued was lowered as it depended on the user having enough tokens and allowance in the case of repayAll(), or having a msg.sender higher than the current debt in the case of repayAllEth().

## Proof of Concept
The affected lines are the following:

https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L147
https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/LendingPair.sol#L156

The scenario for repayAll() is the following:

1. Alice pays off 5 of Bob's Dai debt using repayAll().
2. Bob monitors the mempool for Alice's transaction, and front-runs it by taking out as much debt as Alice's allowance (and therefore balance) to the contract.
3. `debtOf[_token][_account]` now returns the higher amount and pays off Bob's new debt.


The scenario for repayAllEth() is similar:

1. Alice pays off 0.5 of Bob's Weth debt using repayAllEth().
2. Bob monitors the mempool for Alice's transaction, and frontruns it by taking out as much debt as Alice's msg.value amount used.
3. `debtOf[address(WETH)][_account]` now returns the higher amount and pays off Bob's new debt.

## Recommended Mitigation Steps
This issue can be mitigated by enforcing a minimum time to hold debt - e.g. not allowed to repay debt for at least 6 blocks. Alternatively, the repay() function could be used to replace the 2 affected functions by passing in the _amount as the total debt (looked up off-chain and used in the dapp, for example) so that only up to a certain amount of debt is paid. This also means the repay() function would need to be made `payable`, and that the `msg.value` is validated to equal the _amount parameter.

