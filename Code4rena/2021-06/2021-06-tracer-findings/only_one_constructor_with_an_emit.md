## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Only one constructor with an emit](https://github.com/code-423n4/2021-06-tracer-findings/issues/10) 

# Handle

gpersoon


# Vulnerability details

## Impact
The constructor of Insurance.so does an emit.
However the constructors of the other contracts (InsurancePoolToken.sol, Liquidation.sol, Pricing.sol, TracerPerpetualSwaps.sol, TracerPerpetualsFactory.sol, Trader.sol) don't do an emit in the constructor.

## Proof of Concept
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L31
constructor(address _tracer) {
     ...
        emit InsurancePoolDeployed(_tracer, tracer.tracerQuoteToken());
    }

## Tools Used

## Recommended Mitigation Steps
Perhaps it's useful for other constructor to also include an emit


