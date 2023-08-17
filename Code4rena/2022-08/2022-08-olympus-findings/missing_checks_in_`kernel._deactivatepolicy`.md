## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Missing checks in `Kernel._deactivatePolicy`](https://github.com/code-423n4/2022-08-olympus-findings/issues/368) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/Kernel.sol#L325


# Vulnerability details

## Impact
There are no checks to ascertain that the policy being removed is registered in the `Kernel`. Trying to remove a non-registered results in the policy registered at 0th index of `activePolicies` being removed. 

## Proof of Concept
https://github.com/code-423n4/2022-08-olympus/blob/main/src/Kernel.sol#L325

## Recommended Mitigation Steps
Adding `require(activePolicies[idx] == policy_, "Unregistered policy");` will prevent this, where `idx = getPolicyIndex[policy_]`.

**NOTE:** The issue is less likely to happen as this is handled solely by the executor, but having safeguards in the contract is always better than relying on an external factor.
