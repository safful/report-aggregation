## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- filed

# [`flashProof` is not flash-proof](https://github.com/code-423n4/2021-04-vader-findings/issues/218) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The `flashProof` modifier is supposed to prevent flash-loan attacks by disallowing performing several sensitive functions in the same block.

However, it performs this check on `tx.origin` and not on an individual user address basis.
This only prevents flash loan attacks from happening within a single transaction.

But flash loan attacks are theoretically not limited to the same transaction but to the same block as miners have full control of the block and include several vulnerable transactions back to back. (Think transaction _bundles_ similar to flashbot bundles that most mining pools currently offer.)

A miner can deploy a proxy smart contract relaying all contract calls and call it from a different EOA each time bypassing the `tx.origin` restriction.

## Impact

The `flashProof` modifier does not serve its purpose.

## Recommended Mitigation Steps

Try to apply the modifier to individual addresses that interact with the protocol instead of `tx.origin`.

Furthermore, attacks possible with flash loans are usually also possible for whales, making it debatable if adding flash-loan prevention logic is a good practice.


