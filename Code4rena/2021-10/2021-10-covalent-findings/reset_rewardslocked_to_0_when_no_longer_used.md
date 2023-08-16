## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [reset rewardsLocked to 0 when no longer used](https://github.com/code-423n4/2021-10-covalent-findings/issues/13) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _stake() initializes endEpoch using the value of rewardsLocked.
Afterwards rewardsLocked is no longer used (because now endEpoch !=0)

So you can set rewardsLocked to 0 save a bit of gas.

## Proof of Concept
https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L171-L176

## Tools Used

## Recommended Mitigation Steps
Update to code of  _stake() to:
if (endEpoch == 0) {
           endEpoch = uint128(block.number) + rewardsLocked / allocatedTokensPerEpoch;
           rewardsLocked = 0; // no longer used and saves a bit of gas
}

