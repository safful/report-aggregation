## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Misleading comment and missing revert message](https://github.com/code-423n4/2021-07-spartan-findings/issues/31) 

# Handle

jonah1005


# Vulnerability details

## Impact
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L46

The comment at `poolFactory` L46  is a bit misleading.
```
require(getPool(token) == address(0)); // Must be a valid token
```

A similar checks in `synthFactory` seems to be more clear.
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthFactory.sol#L38

```
        require(getSynth(token) == address(0), "exists"); // Synth must not already exist
```

## Proof of Concept
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L46
## Tools Used
None
## Recommended Mitigation Steps

It seems that `PoolFactory` is the only contract that does not provide detailed revert messages. I wonder whether the devs do this because of the concern about the code size limit. If that's the case, I recommend refactoring it to libraries or even uses a proxy factory to create new pools. 

Ref to proxy factory: 
https://eips.ethereum.org/EIPS/eip-1167

