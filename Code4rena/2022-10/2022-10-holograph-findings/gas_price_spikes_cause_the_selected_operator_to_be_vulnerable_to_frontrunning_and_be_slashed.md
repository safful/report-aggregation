## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- selected for report
- edited-by-warden
- responded

# [Gas price spikes cause the selected operator to be vulnerable to frontrunning and be slashed](https://github.com/code-423n4/2022-10-holograph-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L354


# Vulnerability details

## Impact
Gas price spikes cause the selected operator to be vulnerable to frontrunning and be slashed.

## Proof of Concept
```solidity
require(gasPrice >= tx.gasprice, "HOLOGRAPH: gas spike detected");
```

```solidity
        /**
         * @dev select operator that failed to do the job, is slashed the pod base fee
         */
        _bondedAmounts[job.operator] -= amount;
        /**
         * @dev the slashed amount is sent to current operator
         */
        _bondedAmounts[msg.sender] += amount;
```

Since you have designed a mechanism to prevent other operators to slash the operator due to "the selected missed the time slot due to a gas spike". It can induce that operators won't perform their job if a gas price spike happens due to negative profit.

But your designed mechanism has a vulnerability. Other operators can submit their transaction to the mempool and queue it using `gasPrice in bridgeInRequestPayload`. It may get executed before the selected operator as the selected operator is waiting for the gas price to drop but doesn't submit any transaction yet. If it doesn't, these operators lose a little gas fee. But a slashed reward may be greater than the risk of losing a little gas fee.

```solidity
require(timeDifference > 0, "HOLOGRAPH: operator has time");
```

Once 1 epoch has passed, selected operator is vulnerable to slashing and frontrunning.

## Recommended Mitigation Steps
Modify your operator node software to queue transactions immediately with `gasPrice in bridgeInRequestPayload` if a gas price spike happened. Or allow gas fee loss tradeoff to prevent being slashed.