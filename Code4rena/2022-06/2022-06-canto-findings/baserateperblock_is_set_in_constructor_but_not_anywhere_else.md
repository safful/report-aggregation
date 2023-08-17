## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [baseRatePerBlock is set in constructor but not anywhere else](https://github.com/code-423n4/2022-06-canto-findings/issues/72) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/NoteInterest.sol#L73-L77


# Vulnerability details

## Impact
baseRatePerBlock cannot be relied on to accurately contain current interest rate

## Proof of Concept
baseRatePerBlock is set in the constructor but then not update in either updateBaseRate or _setBaseRatePerYear which update baseRatePerYear. Any contract that pulls the interest rate from baseRatePerBlock will always get the interest rate initially set at the creation of the contract

## Tools Used

## Recommended Mitigation Steps
Update baseRatePerBlock in updateBaseRate and _setBaseRatePerYear

