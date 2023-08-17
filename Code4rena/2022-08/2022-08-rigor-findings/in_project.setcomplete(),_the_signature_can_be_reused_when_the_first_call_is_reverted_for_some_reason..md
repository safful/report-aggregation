## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [In Project.setComplete(), the signature can be reused when the first call is reverted for some reason.](https://github.com/code-423n4/2022-08-rigor-findings/issues/263) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L330


# Vulnerability details

## Impact
`setComplete()` function might be called successfully using the past signature when it shouldn't work.

As a result, a task might be completed when a builder doesn't want it.


## Proof of Concept
[approveHash() function](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L108) can set only true so there is no method to cancel already approved hash without [passing validation here](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L891).

So the below scenario would be possible.

- A builder, GC, and SC started a task and SC finished the task.
- They are approved to complete the task and signed the signature.
- But right before to call [setComplete()](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L330) using the signature, the SC felt the cost is too low and raised a dispute to change the order using [raiseDispute()](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L493).
- As I suggested with another medium issue, the task can't be completed when there is an ongoing dispute from [this document - "If there is no ongoing dispute about that project, task status is updated and payment is made."](https://github.com/code-423n4/2022-08-rigor#tasks-completion-and-payment). So `setComplete()` might revert.
- Even if it doesn't check active disputes as now, `setComplete()` might revert when the funds haven't been allocated and a builder signed by fault.
- After that, the HomeFi admin accepted the dispute, and the cost of the task was increased as SC wanted.
- Then the builder would hope to get more results (or scores) from this task as the cost is increased rather than completed right away.
- But SC can call `setComplete()` using the previous signature and complete the task without additional work.
- A builder might know about that before and try to update task hash but it will revert because SC doesn't agree to [updateTaskHash()](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L283).
- In this case, it's logical to cancel the approved hash [here](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L108) but there is no such option.


I don't know if there would be similar problems with other functions that use signature and I think it would reduce the risk a little if we add an option to cancel the approved hash.


## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
Recommend modifying [approveHash()](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L108) like below.

```
function approveHash(bytes32 _hash, bool _bool) external override { //++++++++++++++++++++
    address _sender = _msgSender();
    // Allowing anyone to sign, as its hard to add restrictions here.
    // Store _hash as signed for sender.
    approvedHashes[_sender][_hash] = _bool; //+++++++++++++++++++

    emit ApproveHash(_hash, _sender, _bool); //++++++++++++++++++++++
}
```

I am not so sure that a similar scenario would be possible in the [Community contract](https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Community.sol#L501) also and recommend to change both functions together.