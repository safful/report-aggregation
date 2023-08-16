## Tags

- bug
- duplicate
- sponsor confirmed
- 3 (High Risk)
- resolved

# [Signature replay attacks for different identities (nonce on wrong party)](https://github.com/code-423n4/2021-10-ambire-findings/issues/39) 

# Handle

cmichel


# Vulnerability details

A single `QuickAccount` can serve as the "privilege" for multiple identities, see the comment in `QuickAccManager.sol`:

> NOTE: a single accHash can control multiple identities, as long as those identities set it's hash in privileges[address(this)]. this is by design

If there exist two different identities that _both share the same QuickAccount_ (`identity1.privileges(address(this)) == identity2.privileges(address(this)) == accHash`) the following attack is possible in `QuickAccManager.send`:

Upon observing a valid `send` on the first identity, the same transactions can be replayed on the second identity by an attacker by calling `send` with the same arguments and just changing the `identity` to the second identity.

This is because the `identity` is not part of the `hash`. Including the **nonce of** the identity in the hash is not enough.

Two fresh identities will both take on nonces on zero and lead to the same hash.

## Impact
Transactions on one identity can be replayed on another one if it uses the same `QuickAccount`.
For example, a transaction paying a contractor can be replayed by the contract on the second identity earning the payment twice.

## Recommended Mitigation Steps
1. Nonces should not be indexed by the identity but by the `accHash`. This is because nonces are used to stop replay attacks and thus need to be on the _signer_ (`QuickAccount` in this case), not on the target contract to call.
2. The `identity` _address_ itself needs to be part of `hash` as otherwise the `send` can be frontrun and executed by anyone on the other identity by switching out the `identity` parameter.

## Other occurrences
This issue of using the wrong nonce (on the `identity` which means the nonces repeat per identity) and not including `identity` address leads to other attacks throughout the `QuickAccManager`:
- `cancel`: attacker can use the same signature to cancel the same transactions on the second identity
- `execScheduled`: can frontrun this call and execute it on the second identity instead. This will make the original transaction fail as `scheduled[hash]` is deleted.
- `sendTransfer`: same transfers can be replayed on second identity
- `sendTxns`: same transactions can be replayed on second identity


