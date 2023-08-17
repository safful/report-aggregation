## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- M-08

# [Borrower can cause a DoS by frontrunning a liquidation and repaying as low as 1 wei of the current debt](https://github.com/code-423n4/2023-05-venus-findings/issues/255) 

# Lines of code

https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L469-L472


# Vulnerability details

## Impact
Borrowers can cause DoS when liquidator attempts to liquidate a 100% of the borrower's position.
The borrower just need to frontrun the liquidation tx and repay a slightly portion of the debt, paying as low as 1 wei will make the [borrowBalance](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L446) to be less than what it was when the liquidator sent the tx to liquidate the position.

## Proof of Concept
If aliquidator intends to liquidate the entire position, but the borrower frontruns the liquidator's transaction and repays an insignificant amount of the total debt, will cause the [borrowBalance](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L446) to be less than it was when the liquidator sent its transaction, thus, will cause the value of the [maxClose](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L469) variable to be less than the repayAmount that the liquidator set to liquidate the whole position, which will end up causing the tx to be reverted because of [this validation](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L470-L472)

**Example**
- There is a position of 100 WBNB to be liquidated, the liquidator sends the repayAmount as whatever the `maxClose` was at that point, the borrower realizes that the 100% of its position will be liquidated and then frontruns the liquidation transaction by repaying an insignificant amount of the total borrow.
 - When the liquidation transaction is executed, the maxClose will be calculated based on the new borrowBalance, which will cause the calculation of maxClose to be less than the total repayAmount that was sent, and the transaction will be reverted even though the position is still in a liquidation state


## Tools Used
Manual Audit

## Recommended Mitigation Steps
Instead of [reverting the tx if the repayAmount is greater than maxClose](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol#L469-L472), recalculate the final repayAmount to be paid during the execution of the liquidation and return this calculated value back to the function that called the `preLiquidateHook()`

```solidity
function preLiquidateHook(
    ...
    uint256 repayAmount,
    ...
+   ) external override returns(uint256 repayAmountFinal) {
    ...
    ...
-   if (repayAmount > maxClose) {
-      revert TooMuchRepay();
-   }
+   repayAmountFinal = repayAmount > maxClose ? maxClose : repayAmount
```



## Assessed type

DoS