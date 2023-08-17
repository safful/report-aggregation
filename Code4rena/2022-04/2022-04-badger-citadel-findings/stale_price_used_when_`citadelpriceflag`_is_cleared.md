## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Stale price used when `citadelPriceFlag` is cleared](https://github.com/code-423n4/2022-04-badger-citadel-findings/issues/176) 

# Lines of code

https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/Funding.sol#L430-L437


# Vulnerability details

During the [video](https://drive.google.com/file/d/1hCzQrgZEsbd0t2mtuaXm7Cp3YS-ZIlw3/view?usp=sharing) it was explained that the policy operations team was meant to be a nimble group that could change protocol values considered to be safe. Further, it was explained that since pricing comes from an oracle, and there would have to be unusual coordination between the two to affect outcomes, the group was given the ability to clear the pricing flag to get things moving again once the price was determined to be valid

## Impact
If an oracle price falls out of the valid min/max range, the `citadelPriceFlag` is set to true, but the out-of-bounds value is not stored. If the policy operations team calls `clearCitadelPriceFlag()`, the stale price from before the flag will be used. Not only is it an issue because of stale prices, but this means the policy op team now has a way to affect pricing not under the control of the oracle (i.e. no unusual coordination required to affect an outcome). Incorrect pricing leads to incorrect asset valuations, and loss of funds.

## Proof of Concept

The flag is set but the price is not stored
File: src/Funding.sol (lines [427-437](https://github.com/code-423n4/2022-04-badger-citadel/blob/18f8c392b6fc303fe95602eba6303725023e53da/src/Funding.sol#L427-L437))
```solidity
        if (
            _citadelPriceInAsset < minCitadelPriceInAsset ||
            _citadelPriceInAsset > maxCitadelPriceInAsset
        ) {
            citadelPriceFlag = true;
            emit CitadelPriceFlag(
                _citadelPriceInAsset,
                minCitadelPriceInAsset,
                maxCitadelPriceInAsset
            );
        } else {
```

## Tools Used
Code inspection

## Recommended Mitigation Steps
Always set the `citadelPriceInAsset`


