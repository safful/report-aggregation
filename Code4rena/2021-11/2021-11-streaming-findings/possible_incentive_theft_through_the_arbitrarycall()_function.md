## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Possible incentive theft through the arbitraryCall() function](https://github.com/code-423n4/2021-11-streaming-findings/issues/199) 

# Handle

toastedsteaksandwich


# Vulnerability details

## Impact
The Locke.arbitraryCall() function allows the inherited governance contract to perform arbitrary contract calls within certain constraints. Contract calls to tokens provided as incentives through the createIncentive() function are not allowed if there is some still some balance according to the incentives mapping (See line 735 referenced below). 

However, the token can still be called prior any user creating an incentive, so it's possible for the arbitraryCall() function to be used to set an allowance on an incentive token before the contract has actually received any of the token through createIncentive(). 

In summary:

1) If some possible incentive tokens are known prior to being provided, the arbitraryCall() function can be used to pre-approve a token allowance for a malicious recipient. 
2) Once a user calls createIncentive() and provides one of the pre-approved tokens, the malicious recipient can call transferFrom on the provided incentive token and withdraw the tokens.

## Proof of Concept
https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L735

## Recommended Mitigation Steps

### Recommendation 1
Limit the types of incentive tokens so it can be checked that it's not the target contract for the arbitraryCall().

### Recommendation 2
Validate that the allowance of the target contract (if available) has not changed.

