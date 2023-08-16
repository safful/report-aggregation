## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Long Revert Strings](https://github.com/code-423n4/2021-10-ambire-findings/issues/16) 

# Handle

ye0lde


# Vulnerability details

## Impact

Shortening revert strings to fit in 32 bytes will decrease deploy time gas and will decrease runtime gas when the revert condition has been met.  

Revert strings that are longer than 32 bytes require at least one additional mstore, along with additional overhead for computing memory offset, etc.

## Proof of Concept

Revert strings > 32 bytes are here:
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/libs/SignatureValidatorV2.sol#L33
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/libs/SignatureValidatorV2.sol#L55
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/libs/SignatureValidatorV2.sol#L59
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/libs/SignatureValidatorV2.sol#L64
https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/libs/SignatureValidatorV2.sol#L65

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Shorten the revert strings to fit in 32 bytes.

