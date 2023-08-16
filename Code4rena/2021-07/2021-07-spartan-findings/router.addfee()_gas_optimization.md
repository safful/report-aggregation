## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ROUTER.addFee() Gas Optimization](https://github.com/code-423n4/2021-07-spartan-findings/issues/80) 

# Handle

natus


# Vulnerability details

## Impact 
This function calls feeArray from storage when setting 'n' and then twice every loop (feeArray[i] && feeArray[i - 1]) and then again after the loop once.
 
If this was instead called once at the start and stored in memory; iterated and then assigned into the storage at the end; could save some gas
 
## Proof of Concept
UTILS lines #300 to #306

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L301 
 
## Recommended Mitigation Steps
Step1: Above line #301; add in:
uint [] memory _feeArray = feeArray
(Get the storage feeArray and store it in memory)

Step2: change feeArray.length to _feeArray.length

Step3: change:
feeArray[i] = feeArray[i - 1]
to:
_feeArray[i] = _feeArray[i - 1]

Step4: change:
feeArray[0] = _fee
To:
_feeArray[0] = _fee

Step5: add:
feeArray = _feeArray 
as the final line inside the function

