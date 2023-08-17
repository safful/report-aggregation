## Tags

- bug
- enhancement
- 2 (Med Risk)
- sponsor confirmed

# [possible DoS on vestingRecipients due to lack of disposal mechanism](https://github.com/code-423n4/2022-09-vtvl-findings/issues/128) 

# Lines of code

https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L224
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L245
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L302
https://github.com/code-423n4/2022-09-vtvl/blob/f68b7f3e61dad0d873b5b5a1e8126b839afeab5f/contracts/VTVLVesting.sol#L317


# Vulnerability details

## Impact
- L224/245/302/317 - When the smart contracts start to be used, the variable in storage vestingRecipients will start to be filled with addresses, as there is no mechanism to eliminate elements, this will cause the allVestingRecipients() function to generate a DoS yes has many addressess.


## Recommended Mitigation Steps
In the withdraw() function you could remove the element from vestingRecipients that no longer has vesting. This would make the variable not grow without reducing elements.
