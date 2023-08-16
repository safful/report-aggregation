## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity
- resolved

# [gracePeriod not increased after membership extension](https://github.com/code-423n4/2021-05-fairside-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function purchaseMembership of FSDNetwork.sol, when the membership is extended then membership[msg.sender].creation is increased, however 
membership[msg.sender].gracePeriod is not increased.
This might lead to a gracePeriod than is less then expected.
It seems logical to also increase the gracePeriod  

## Proof of Concept
FSDNetwork.sol
// https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/network/FSDNetwork.sol#L171
function purchaseMembership(uint256 costShareBenefit) external {
     ...
      if (membership[msg.sender].creation == 0) {
            ...
            membership[msg.sender].creation       = block.timestamp;
            membership[msg.sender].gracePeriod =  membership[msg.sender].creation +  MEMBERSHIP_DURATION +  60 days;
        } else {
          ....
          membership[msg.sender].creation += durationIncrease;
   }

## Tools Used
Editor

## Recommended Mitigation Steps
Check if gracePeriod has to be increased also. 
When that is the case add the logic to do that.

