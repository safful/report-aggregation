## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Griefing/DOS of withdrawals by EOAs from treasury (TRSRY) possible](https://github.com/code-423n4/2022-08-olympus-findings/issues/317) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/TreasuryCustodian.sol#L53-L67


# Vulnerability details

## Impact
Any withdrawals from the treasury by an approved EOA can be denied by a malicious actor that watches the mempool.

## Proof of Concept
The function TreasuryCustodian.revokePolicyApprovals() doesnt provide sufficient checks for its intended purpose of "revoking a deactivated policy's approvals". As can be seen by the TODO labels, the issue has already been acknowledged by the team (regardless it is still an issue present in an in-scope contract). The only check performed is trying to call the isActive()-function on an address and interpret the returned value as boolean. Attempting to call this function on an EOA will not fail and return 0 (=false). Hence the condition to revert is not fulfilled and the amounts approved to withdraw will be set to 0. 

## Tools Used

IDE (Remix, VSCode)

## Recommended Mitigation Steps

A partial but insufficient fix would be to check if the address passed to the function contains code and hence is not an EOA. A better approach might be to add a mapping(address => bool) of all addresses that have been active policies some time in the past to the kernel, something like this:

As a public variable in Kernel.sol
`mapping(address => bool) public isRegisteredPolicy;`

in Kernel.activatePolicy():
`isRegisteredPolicy[address(policy_)] ) = true;`

and finally in TreasuryCustodian.revokePolicyApprovals():
`if(!kernel.isRegisteredPolicy(policy_) revert NotARegisteredPolicy`