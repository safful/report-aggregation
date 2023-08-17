## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [updateProjectHash does not check project address](https://github.com/code-423n4/2022-08-rigor-findings/issues/347) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L162


# Vulnerability details

In Project.sol, function `updateProjectHash` L162, `_data` (which is signed by builder and/or contractor) does not contain a reference to the project address. In all other external functions of Project.sol, `_data` contains the address of the project, used in this check: 
```require(_projectAddress == address(this), "Project::!projectAddress");```.
The lack of this verification makes it possible to reuse the same `_data`, and the same `_signature` on another project, in the case the latter has the same builder and/or contractor, and the same `_nonce`. In pratice, if the same group of people starts a new project, when `_nonce` reaches the correct value, anyone can change the hash of a task (if we suppose that that `updateTaskHash()` was used in the previous project).