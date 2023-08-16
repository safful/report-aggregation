## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Input validation not done in few important functions in Parameters.sol](https://github.com/code-423n4/2022-01-insure-findings/issues/243) 

# Handle

hubble


# Vulnerability details


## Impact
Input validation required for few important parameters as mentioned in the below functions.

## Proof of Concept
File : Parameters.sol
   line 120 :     function setUpperSlack(address _address, uint256 _target)  
        Need to check that the _target value should be less than or equal to 100% (1000)

   line 134 :     function setLowerSlack(address _address, uint256 _target) 
        Need to check that the _target value should be less than or equal to corresponding UpperSlack Value

   line 177 :     function setFeeRate(address _address, uint256 _target)  
        Need to check that the _target value should be less than or equal to 1e6 (1000000)

   line 191 :     function setMaxList(address _address, uint256 _target)  
        Need to check that the _target value should be greater than 1

## Tools Used
Manual review

## Recommended Mitigation Steps
Add require statements with proper value and comments for the respective input fields as given above


