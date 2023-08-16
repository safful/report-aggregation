## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Remove _token from addLiquiditySingleForMember](https://github.com/code-423n4/2021-07-spartan-findings/issues/64) 

# Handle

natus


# Vulnerability details

## Impact 
Gas optimization / non-critical issue. Wasted lines of code creating and setting a local variable that does not appear to be required. I can't think of a reason to leave it in. 
 
## Proof of Concept
_token local variable is not used anywhere in the codebase. Can be removed to save gas and compile size. Looks like it's to handle WBNB (if it was required) but forgot to check/remove after it was catered for elsewhere.
- https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L83 
 
## Tools Used
N/A 
 
## Recommended Mitigation Steps
ROUTER.addLiquiditySingleForMember()
- Remove line #82
- Remove line #83

