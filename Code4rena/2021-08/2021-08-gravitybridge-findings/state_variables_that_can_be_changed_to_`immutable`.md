## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [State Variables that can be changed to `immutable`](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/48) 

# Handle

hrkrshnn


# Vulnerability details

## State Variables that can be changed to `immutable`

[Solidity 0.6.5](https://blog.soliditylang.org/2020/04/06/solidity-0.6.5-release-announcement/)
introduced `immutable` as a major feature. It allows setting
contract-level variables at construction time which gets stored in code
rather than storage.

Consider the following generic example:

``` solidity
contract C {
    /// The owner is set during contruction time, and never changed afterwards.
    address public owner = msg.sender;
}
```

In the above example, each call to the function `owner()` reads from
storage, using a `sload`. After
[EIP-2929](https://eips.ethereum.org/EIPS/eip-2929), this costs 2100 gas
cold or 100 gas warm. However, the following snippet is more gas
efficient:

``` solidity
contract C {
    /// The owner is set during contruction time, and never changed afterwards.
    address public immutable owner = msg.sender;
}
```

In the above example, each storage read of the `owner` state variable is
replaced by the instruction `push32 value`, where `value` is set during
contract construction time. Unlike the last example, this costs only 3
gas.

### Examples

1.  <https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L59>
2.  <https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/solidity/contracts/Gravity.sol#L60>



