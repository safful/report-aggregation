## Tags

- bug
- 2 (Med Risk)
- primary issue
- sponsor confirmed
- selected for report
- responded

# [Bond tokens (HLG) can get permanently stuck in operator](https://github.com/code-423n4/2022-10-holograph-findings/issues/322) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L374-L382
https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L849-L857


# Vulnerability details

## Impact
Bond tokens (HLG) equal to the slash amount will get permanently stuck in the HolographOperator each time a job gets executed by someone who is not an (fallback-)operator.

## Proof of Concept
The `HolographOperator.executeJob` function can be executed by anyone after a certain passage of time:

```js
...
if (job.operator != address(0)) {
    ...
    if (job.operator != msg.sender) {
        //perform time and gas price check
        if (timeDifference < 6) {
            // check msg.sender == correct fallback operator
        }
        // slash primary operator
        uint256 amount = _getBaseBondAmount(pod);
        _bondedAmounts[job.operator] -= amount;
        _bondedAmounts[msg.sender] += amount;

        //determine if primary operator retains his job
        if (_bondedAmounts[job.operator] >= amount) {
            ...
        } else {
            ...
        }
    }
}
// execute the job
```
In case `if (timeDifference < 6) {` gets skipped, the slashed amount will be assigned to the `msg.sender` regardless if that sender is currently an operator or not. The problem lies within the fact that if `msg.sender` is not already an operator at the time of executing the job, he cannot become one after, to retrieve the reward he got for slashing the primary operator. This is because the function `HolographOperator.bondUtilityToken` requires `_bondedAmounts` to be 0 prior to bonding and hence becoming an operator:

```js
require(_bondedOperators[operator] == 0 && _bondedAmounts[operator] == 0, "HOLOGRAPH: operator is bonded");
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Assuming that it is intentional that non-operators can execute jobs (which could make sense, so that a user could finish a bridging process on his own, if none of the operators are doing it): remove the requirement that `_bondedAmounts` need to be 0 prior to bonding and becoming an operator so that non-operators can get access to the slashing reward by unbonding after.

Alternatively (possibly preferrable), just add a method to withdraw any `_bondedAmounts` of non-operators.