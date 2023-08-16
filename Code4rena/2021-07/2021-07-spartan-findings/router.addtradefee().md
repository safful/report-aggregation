## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ROUTER.addTradeFee()](https://github.com/code-423n4/2021-07-spartan-findings/issues/86) 

# Handle

natus


# Vulnerability details

## Impact    
This is called with every dividend-generating txn (which is 100 or so txns per day/era by default and can be cranked up with an increase in txn volume, so higher importance than some of the other gas opts) 
    
This is a little harder to optimize as it uses the changed state within the same function; however there is still room for optimization despite that; see below. 

arrayFeeSize is called from storage twice every time; and if the array is fully built it's also called another 20 times (by default; can be increased by dao) per call to this function. As this variable doesn't change within this function; it can simply be called once at the started and stored in memory, should be a decent gas opt in a very commonly occuring txn

feeArray is also called from storage only once whilst the array is still building and not complete (this is okay) but once it's built it's called 20 times (again; by default; this might be raised) If we instead call this from storage once *just before* it's required in the loop (has to be after addFee() as this changes that feeArray's state) we can save even more gas

also; arrayFeeLength does not need to be stored in memory; just use feeArray.length from torage instead (only used once, so will only save the memory storage gas which is small)
    
## Proof of Concept   
ROUTER lines #285 to #297
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L285
    
## Recommended Mitigation Steps   
Step1: add at the start of the function:
uint _arrayFeeSize = arrayFeeSize
(Get storage arrayFeeSize & store in memory)

Step2: Replace all 3 instances of arrayFeeSize with _arrayFeeSize

Step3: add below addFee(_fee):
uint [] memory _feeArray = feeArray
(Get storage feeArray & store in memory)

Step4: replace feeArray[i] (inside the loop) to _feeArray[i]

Step5: remove line:
uint arrayFeeLength = feeArray.length
and replace arrayFeeLength with feeArray.length; no need to store in memory if its only used once

