## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Business logic bug in __abdicate() function - 2 Bugs](https://github.com/code-423n4/2021-11-streaming-findings/issues/132) 

# Handle

cyberboy


# Vulnerability details

## Impact
The __abdicate() function at https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L46-L50 is the logic to remove the governance i.e., to renounce governance. However, the function logic does not consider emergency governor and pending governor, which can be a backdoor as only the "gov" is set to zero address while the emergency and pending gov remains. A pending gov can just claim and become the gov again, replacing the zero address.

## Proof of Concept
1. Compile the contract and set the _GOVERNOR and _EMERGENCY_GOVERNOR.
2. Now set a pendingGov but do not call acceptGov()

Bug 1
3. Call the __abdicate() function and we will notice only "gov" is set to zero address while emergency gov remains.

Bug2
4. Now use the address used in "pendingGov" to call acceptGov() function.
5. We will notice the new gov has been updated to the new address from the zero address. 

Hence the __abdicate() functionality can be used as a backdoor using emergency governor or leaving a pending governor to claim later. 



## Tools Used
Remix to test the poC

## Recommended Mitigation Steps
The __abdicate() function should set emergency_gov and pendingGov as well to zero address. 

