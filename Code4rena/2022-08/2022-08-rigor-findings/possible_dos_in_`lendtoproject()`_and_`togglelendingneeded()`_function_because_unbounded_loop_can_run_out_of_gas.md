## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method
- valid

# [Possible DOS in `lendToProject()` and `toggleLendingNeeded()` function because unbounded loop can run out of gas](https://github.com/code-423n4/2022-08-rigor-findings/issues/336) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L710


# Vulnerability details

## Impact

In `Project` contract, the `lendToProject()` function might not be available to be called if there are a lot of Task in `tasks[]` list of project. It means that the project cannot be funded by either builder or community owner.

This can happen because `lendToProject()` used `projectCost()` function. And the loop in `projectCost()` did not have a mechanism to stop, it’s only based on the length `taskCount`, and may take all the gas limit. If the gas limit is reached, this transaction will fail or revert.

Same issue with `toggleLendingNeeded()` function which also call `projectCost()` function.

## Proof of Concept

Function `projectCost()` did not have a mechanism to stop, only based on the `taskCount`.
```solidity
function projectCost() public view override returns (uint256 _cost) {
    // Local instance of taskCount. To save gas.
    uint256 _length = taskCount;

    // Iterate over all tasks to sum their cost
    for (uint256 _taskID = 1; _taskID <= _length; _taskID++) {
        _cost += tasks[_taskID].cost;
    }
}
```

There is no limit for builder when [add task](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L248-L257).

And function `lendToProject()` used `projectCost()` to [check the new total lent value](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L199-L202)
```solidity
require(
    projectCost() >= uint256(_newTotalLent),
    "Project::value>required"
);
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Consider keep value of `projectCost()` in a storage variable and update it when a task is added or updated accordingly.

