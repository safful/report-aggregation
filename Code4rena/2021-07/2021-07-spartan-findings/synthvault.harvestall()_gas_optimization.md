## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SYNTHVAULT.harvestAll() Gas Optimization](https://github.com/code-423n4/2021-07-spartan-findings/issues/82) 

# Handle

natus


# Vulnerability details

## Impact   
Here we call the storage stakedSynthAssets 3 times in the loop or 4 times per loop if the reward is > 0. 

It could instead be called once before the loop and stored in memory. Will save more gas as time goes on and the stakedSynthAssets array potentially gets larger as more assets get listed 
   
## Proof of Concept  
SYNTHVAULT lines #121 to #132  
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthVault.sol#L121 
   
## Recommended Mitigation Steps  
Step1: Above line #122; add in:  
address [] memory _stakedSynthAssets = stakedSynthAssets 
(Get the storage stakedSynthAssets and store it in memory)  
 
Then: replace all 4 instances of: 
stakedSynthAssets with _stakedSynthAssets

