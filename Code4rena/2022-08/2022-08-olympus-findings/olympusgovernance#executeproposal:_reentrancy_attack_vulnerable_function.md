## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [OlympusGovernance#executeProposal: reentrancy attack vulnerable function](https://github.com/code-423n4/2022-08-olympus-findings/issues/132) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Governance.sol#L265
https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Governance.sol#L278-L288


# Vulnerability details

## Impact
Given that the activeProposal change is done before the for loop, if this function is call through one kernel.executeAction(instruction,target) we can call the same instructions (in the same order) again and again, which may or may not affect funds (depending on the instructions).

## Proof of Concept
For instance, if we install a new module, and this module has a vulnerability (even intentional), the next steps can by trigger:

1. Call executeAction
1. This allow us to call kernel.executeAction in the for loop
1. executAction allow us to call **_installModule**
1. **\_installModule** allow us to call **newModule_.Init**
1. By init we can call now executeProposal again (suppose that the init function interact with a previous vulnerable proxy contract to scam voters to vote in favour of this proposal as if it was a contract which is ok, and before calling executeProposal we change the implementation to allow this attack),

## Tools Used
Static Analysis

## Recommended Mitigation Steps
Use nonReentrant modifier or move the line ```activeProposal = ActivatedProposal(0, 0);``` before the for loop.
