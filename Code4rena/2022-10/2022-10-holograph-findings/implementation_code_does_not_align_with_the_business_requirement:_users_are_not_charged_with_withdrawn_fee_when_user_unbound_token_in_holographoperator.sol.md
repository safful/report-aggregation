## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [Implementation code does not align with the business requirement: Users are not charged with withdrawn fee when user unbound token in HolographOperator.sol](https://github.com/code-423n4/2022-10-holograph-findings/issues/142) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L899
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L920
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L924
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L928
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L932


# Vulnerability details

## Impact

When user call unbondUtilityToken to unstake the token, 

the function read the available bonded amount, and transfer back to the operator

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L899

```solidity
/**
 * @dev get current bonded amount by operator
 */
uint256 amount = _bondedAmounts[operator];
/**
 * @dev unset operator bond amount before making a transfer
 */
_bondedAmounts[operator] = 0;
/**
 * @dev remove all operator references
 */
_popOperator(_bondedOperators[operator] - 1, _operatorPodIndex[operator]);
/**
 * @dev transfer tokens to recipient
 */
require(_utilityToken().transfer(recipient, amount), "HOLOGRAPH: token transfer failed");
```

the logic is clean, but does not conform to the buisness requirement in the documentation, the doc said

https://docs.holograph.xyz/holograph-protocol/operator-network-specification#operator-job-selection

>To move to a different pod, an Operator must withdraw and re-bond HLG. Operators who withdraw HLG will be charged a 0.1% fee, the proceeds of which will be burned or returned to the Treasury.

The charge 0.1% fee is not implemented in the code.

there are two incentive for bounded operator to stay,

the first is the reward incentive, the second is to avoid penalty with unbonding.

Without chargin the unstaking fee, the second incentive is weak and the operator can unbound or bond whenver they want

## Proof of Concept

https://docs.holograph.xyz/holograph-protocol/operator-network-specification#operator-job-selection

## Tools Used

Manual Review

## Recommended Mitigation Steps

we recommend charge the 0.1% unstaking fee to make the code align with the busienss requirement in the doc.

```solidity
/**
 * @dev get current bonded amount by operator
 */
uint256 amount = _bondedAmounts[operator];
uint256 fee = chargedFee(amount); // here
amount -= fee;  
/**
 * @dev unset operator bond amount before making a transfer
 */
_bondedAmounts[operator] = 0;
/**
 * @dev remove all operator references
 */
_popOperator(_bondedOperators[operator] - 1, _operatorPodIndex[operator]);
/**
 * @dev transfer tokens to recipient
 */
require(_utilityToken().transfer(recipient, amount), "HOLOGRAPH: token transfer failed");
```