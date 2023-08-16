## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Use pragma abicoder v2](https://github.com/code-423n4/2021-08-notional-findings/issues/5) 

# Handle

gpersoon


# Vulnerability details

## Impact
The code contains "pragma experimental ABIEncoderV2;"
In the later Solidity versions it is no longer necessary to use the "experimental" version.
Using experimental constructions is not recommended for production code.

See: https://docs.soliditylang.org/en/v0.8.7/layout-of-source-files.html#abiencoderv2

## Proof of Concept
https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/Router.sol
```JS
pragma experimental ABIEncoderV2;
```

## Tools Used

## Recommended Mitigation Steps
Replace
pragma experimental ABIEncoderV2;
with
pragma abicoder v2;

And make sure you use at least solidity version 0.7.5



