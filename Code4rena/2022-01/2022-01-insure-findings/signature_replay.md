## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Signature replay](https://github.com/code-423n4/2022-01-insure-findings/issues/184) 

# Handle

0x1f8b


# Vulnerability details

## Impact
Signature replay in `PoolTemplate`.

## Proof of Concept
The `redeem` method of `PoolTemplate` verifies the data stored in `incident`, and the verification logic of this process is performed as following:

```
require(
            MerkleProof.verify(
                _merkleProof,
                _targets,
                keccak256(
                    abi.encodePacked(_insurance.target, _insurance.insured)
                )
            ) ||
                MerkleProof.verify(
                    _merkleProof,
                    _targets,
                    keccak256(abi.encodePacked(_insurance.target, address(0)))
                ),
            "ERROR: INSURANCE_EXEMPTED"
        );
```

As can be seen, the only data related to the `_insurance` are` target` and `insured`, so as the incident has no relation with the` Insurance`, apparently nothing prevents a user to call `insure` with high amounts, after receive the incident, the only thing that prevents this from being reused is that the owner creates the incident with an `_incidentTimestamp` from the past.

So if a owner create a incident from the future it's possible to create a new `insure` that could be reused by the same affected address.

Another lack of input verification that could facilitate this attack is the `_span=0` in the `insure` method.

## Tools Used
Manual review.

## Recommended Mitigation Steps
It is mandatory to add a check in `applyCover` that` _incidentTimestamp` is less than the current date and the `span` argument is greater than 0 in the` insure` method.

