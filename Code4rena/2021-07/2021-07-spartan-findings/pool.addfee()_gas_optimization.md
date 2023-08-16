## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [POOL.addFee() Gas Optimization](https://github.com/code-423n4/2021-07-spartan-findings/issues/81) 

# Handle

natus


# Vulnerability details

## Impact  
This function calls revenueArray from storage when setting 'n' and then twice every loop (revenueArray[i] && revenueArray[i - 1]) and then again after the loop once. 
  
If this was instead called once at the start and stored in memory; iterated and then assigned into the storage at the end; could save some gas 
  
## Proof of Concept 
POOL lines #357 to #363
 
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L357 
  
## Recommended Mitigation Steps 
Step1: Above line #358; add in: 
uint [] memory _revArray = revenueArray
(Get the storage revenueArray and store it in memory) 
 
Step2: change revenueArray.length to _revArray.length 
 
Step3: change: 
revenueArray[i] = revenueArray[i - 1] 
to: 
_revArray[i] = _revArray[i - 1] 
 
Step4: change: 
revenueArray[0] = _fee 
To: 
_revArray[0] = _fee 
 
Step5: add: 
revenueArray = _revArray
as the final line inside the function

