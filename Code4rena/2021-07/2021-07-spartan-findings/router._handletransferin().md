## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ROUTER._handleTransferIn()](https://github.com/code-423n4/2021-07-spartan-findings/issues/87) 

# Handle

natus


# Vulnerability details

## Impact
Here we return a value that isnt used anywhere which can safely be removed. This will save the return and also memory store gas costs

If the asset is not BNB/WBNB; we also get the startBal which is another memory store that isnt required, but more importantly we do a more expensive call to check the balance of the token in the pool contract which isnt required. This goes a step further at the 'actual' step at the end where we call the balance again and then do a MINUS math operation calling the memory value

Removing those lines will make all transactions cheaper that involve moving assets through the ROUTER, which appears to be quite a lot and sometimes even multiple times per function
 
## Proof of Concept    
ROUTER lines #197 to #211 
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L197 
     
## Recommended Mitigation Steps    
Remove these lines:
#204
#206
#208

Also remove the return:
returns(uint256 actual)

