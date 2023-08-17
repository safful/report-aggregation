## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- selected for report
- responded

# [Wrong slashing calculation rewards for operator that did not do his job](https://github.com/code-423n4/2022-10-holograph-findings/issues/307) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L374-L382


# Vulnerability details

## Impact

Wrong slashing calculation may create unfair punishment for operators that accidentally forgot to execute their job.

## Proof of Concept

[Docs](https://docs.holograph.xyz/holograph-protocol/operator-network-specification): If an operator acts maliciously, a percentage of their bonded HLG will get slashed. Misbehavior includes (i) downtime, (ii) double-signing transactions, and (iii) abusing transaction speeds. 50% of the slashed HLG will be rewarded to the next operator to execute the transaction, and the remaining 50% will be burned or returned to the Treasury.

The docs also include a guide for the number of slashes and the percentage of bond slashed. However, in the contract, there is no slashing of percentage fees. Rather, the whole _getBaseBondAmount() fee is [slashed from the job.operator instead.](https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L374-L382)

```
        uint256 amount = _getBaseBondAmount(pod);
        /**
         * @dev select operator that failed to do the job, is slashed the pod base fee
         */
        _bondedAmounts[job.operator] -= amount;
        /**
         * @dev the slashed amount is sent to current operator
         */
        _bondedAmounts[msg.sender] += amount;
```

Documentation states that only a portion should be slashed and the number of slashes should be noted down. 

## Tools Used

Manual Review

## Recommended Mitigation Steps

Implement the correct percentage of slashing and include a mapping to note down the number of slashes that an operator has