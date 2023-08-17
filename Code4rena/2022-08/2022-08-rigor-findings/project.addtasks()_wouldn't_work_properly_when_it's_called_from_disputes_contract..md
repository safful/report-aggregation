## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Project.addTasks() wouldn't work properly when it's called from disputes contract.](https://github.com/code-423n4/2022-08-rigor-findings/issues/233) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L238


# Vulnerability details

## Impact
`addTasks()` function checks [this require()](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L238) to make sure `_taskCount` is correct.

But it might revert when this function is called after a dispute because it takes a certain time to resolve disputes and other tasks might be added meanwhile.


## Proof of Concept
The below scenario would be possible.

- A project contains 10 active tasks(taskCount = 10) and a builder and contractor are going to add one more task.
- There were some disagreements between a builder and contractor so they raised a dispute with _taskCount = 10 using [raiseDispute()](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L493).
- Normally it would take a certain time(like 1 day or more) to resolve the dispute as it must be done by HomeFi owner.
- Meanwhile, if the builder and contractor need to add another task, they should set `_taskCount = 10` and `taskCount` will be 11 after addition [here](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L260).
- After that, the HomeFi admin agreed to add a task with `_taskCount = 10`, but it will revert [here](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L238).

So currently, the project builder and contractor shouldn't add new tasks to make their previous dispute valid.

I think it's reasonable to modify that they can add other tasks even though there is an active dispute.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
I think we can modify not to compare [taskCount](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L238) when it's called from disputes contract.

So we can modify [this part](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L238) like below.


```
if (_msgSender() != disputes) {
    require(_taskCount == taskCount, "Project::!taskCount");
}
else {
    _taskCount = taskCount;
}
```