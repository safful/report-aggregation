## Tags

- bug
- 2 (Med Risk)
- primary issue
- sponsor confirmed
- selected for report
- responded

# [ Attacker can force chaotic operator behavior](https://github.com/code-423n4/2022-10-holograph-findings/issues/432) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/HolographOperator.sol#L875


# Vulnerability details

## Description

Operators are organized into different pod tiers. Every time a new request arrives, it is scheduled to a random available pod. It is important to note that pods may be empty, in which case the pod array actually has a single zero element to help with all sorts of bugs. When a pod of a non existing tier is created, any intermediate tiers between the current highest tier to the new tier are filled with zero elements. This happens at bondUtilityToken():

```
if (_operatorPods.length < pod) {
  /**
   * @dev activate pod(s) up until the selected pod
   */
  for (uint256 i = _operatorPods.length; i <= pod; i++) {
    /**
     * @dev add zero address into pod to mitigate empty pod issues
     */
    _operatorPods.push([address(0)]);
  }
}
```

The issue is that any user can spam the contract with a large amount of empty operator pods. The attack would look like this:

1. bondUtilityToken(attacker, large_amount, high_pod_number)
2. unbondUtilityToken(attacker, attacker)

The above could be wrapped in a flashloan to get virtually any pod tier filled.

The consequence is that when the scheduler chooses pods uniformally, they will very likely choose an empty pod, with the zero address. Therefore, the chosen operator will be 0, which is referred to in the code as "open season". In this occurrance, any operator can perform the executeJob() call. This is of course really bad, because all but one operator continually waste gas for executions that will be reverted after the lucky first transaction goes through. This would be a practical example of a griefing attack on Holograph.


## Impact

Any user can force chaotic "open season" operator behavior

## Tools Used

Manual audit

## Recommended Mitigation Steps

It is important to pay special attention to the scheduling algorithm, to make sure different pods are given execution time according to the desired heuristics.