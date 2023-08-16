## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Overwrite benRevocable](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/132) 

# Handle

gpersoon


# Vulnerability details

## Impact
Anyone can call the function vest() of Vesting.sol, for example with a smail "_amount" of tokens, for any _beneficiary.

The function overwrites the value of benRevocable[_beneficiary], effectively erasing any previous value.

So you can set any _beneficiary to Revocable.
Although revoke() is only callable by the owner, this is circumventing the entire mechanism of benRevocable.

## Proof of Concept
// https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/Vesting.sol#L73-L98

function vest(address _beneficiary, uint256 _amount, uint256 _isRevocable) external payable whenNotPaused {
        ...
        if(_isRevocable == 0){
            benRevocable[_beneficiary] = [false,false];  // just overwrites the value
        }
        else if(_isRevocable == 1){
            benRevocable[_beneficiary] = [true,false]; // just overwrites the value
        }      

## Tools Used

## Recommended Mitigation Steps
Whitelist the calling of vest()
Or check if values for benRevocable are already set.


