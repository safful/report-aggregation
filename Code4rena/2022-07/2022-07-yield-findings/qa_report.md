## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed
- old-submission-method

# [QA Report](https://github.com/code-423n4/2022-07-yield-findings/issues/174) 

## Wrong Comment in `SetLimit` Function

There is a wrong comment in `SetLimit` function. The comment doesn't explain the parameters used in the function. this can confuse the reader when reading the code. The comment is located below:
https://github.com/code-423n4/2022-07-yield/blob/main/contracts/Witch.sol#L118-L122