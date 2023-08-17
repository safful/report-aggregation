## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Users can lose fractions to precision loss during migraction if _newFractionSupply is set very low](https://github.com/code-423n4/2022-07-fractional-findings/issues/137) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L72-L99


# Vulnerability details

# Vulnerability details

## Impact
Precision loss causing loss of user value and potentially cause complete loss to vault

## Proof of Concept
https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L471-L472

If the supply of the fraction is set to say 10 then any user that uses migrateFractions with less than 10% of the contributions will receive no shares at all due to precision loss. Under certain conditions it may even cause complete loss of access to the vault. In this same example, if less than 5 fractions can be redeemed (i.e. not enough people have more than 10% to overcome the precision loss) then the vault would never be able to be bought out and the vault would forever be frozen.

## Tools Used

## Recommended Mitigation Steps
When calling propose require that _newFractionSupply is greater than some value (i.e. 1E18)

