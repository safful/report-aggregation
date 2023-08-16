## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- EmergencyBrake
- Timelock

# [Timelock.sol: Indexing targets array might not be useful](https://github.com/code-423n4/2021-08-yield-findings/issues/63) 

# Handle

hickuphh3


# Vulnerability details

### Impact

As per the [solidity documentation](https://docs.soliditylang.org/en/v0.8.1/abi-spec.html?highlight=events#events):

However, for all “complex” types or types of dynamic length, including all arrays, string, bytes and structs, EVENT_INDEXED_ARGS will contain the Keccak hash of a special in-place encoded value (see Encoding of Indexed Event Parameters), rather than the encoded value directly.

It therefore might not be useful to index `address[] targets` in `Timelock.sol` and `address[] contacts` in `EmergencyBrake.sol`, since it's the keccak hash of the addresses. [Also saves gas to drop the indexed keyword](https://ethereum.stackexchange.com/questions/56486/does-it-make-a-difference-to-index-an-event-with-one-parameter/56491).

### Recommended Mitigation Steps

Remove the `indexed` keyword for the arguments mentioned above.

