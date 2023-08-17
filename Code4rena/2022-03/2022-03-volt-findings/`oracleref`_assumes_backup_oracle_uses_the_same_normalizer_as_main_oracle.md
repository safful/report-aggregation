## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`OracleRef` assumes backup oracle uses the same normalizer as main oracle](https://github.com/code-423n4/2022-03-volt-findings/issues/26) 

# Lines of code

https://github.com/code-423n4/2022-03-volt/blob/f1210bf3151095e4d371c9e9d7682d9031860bbd/contracts/refs/OracleRef.sol#L104


# Vulnerability details

## Impact
The `OracleRef` assumes that the backup oracle uses the same normalizer as the main oracle.
This generally isn't the case as it could be a completely different oracle, not even operated by Chainlink.

If the main oracle fails, the backup oracle could be scaled by a wrong amount and return a wrong price which could lead to users being able to mint volt cheap or redeem volt for inflated underlying amounts.

## Recommended Mitigation Steps
Should there be two scaling factors, one for each oracle?


