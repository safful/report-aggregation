## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- selected for report

# [After proposed 0.8.0 upgrade kicks in, L2 finalizeInboundTransfer might not work.](https://github.com/code-423n4/2022-10-thegraph-findings/issues/289) 

# Lines of code

https://github.com/code-423n4/2022-10-thegraph/blob/309a188f7215fa42c745b136357702400f91b4ff/contracts/l2/gateway/L2GraphTokenGateway.sol#L70


# Vulnerability details

## Description

L2GraphTokenGateway uses the onlyL1Counterpart modifier to make sure finalizeInboundTransfer is only called from L1GraphTokenGateway. Its implementation is:

```Solidity
modifier onlyL1Counterpart() {
        require(
            msg.sender == AddressAliasHelper.applyL1ToL2Alias(l1Counterpart),
            "ONLY_COUNTERPART_GATEWAY"
        );
        _;
    }
```

It uses applyL1ToL2Alias defined as:

```
uint160 constant offset = uint160(0x1111000000000000000000000000000000001111);

    /// @notice Utility function that converts the address in the L1 that submitted a tx to
    /// the inbox to the msg.sender viewed in the L2
    /// @param l1Address the address in the L1 that triggered the tx to L2
    /// @return l2Address L2 address as viewed in msg.sender
    function applyL1ToL2Alias(address l1Address) internal pure returns (address l2Address) {
        l2Address = address(uint160(l1Address) + offset);
    }
```

This behavior matches with how Arbitrum augments the sender's address to L2. The issue is that I've spoken with the team and they are [planning](https://github.com/graphprotocol/contracts/pull/725) an upgrade from Solidity 0.7.6 to 0.8.0. Their proposed [changes](https://github.com/graphprotocol/contracts/blob/c4d3cb56cb4032dbb3a0f1b7535b5d94ccf86222/contracts/arbitrum/AddressAliasHelper.sol) will break this function, because under 0.8.0, this line has a ~ 1/15 chance to overflow:

`l2Address = address(uint160(l1Address) + offset);`

Interestingly, the sum intentionally wraps around using the uint160 type to return a correct address, but this wrapping will overflow in 0.8.0

## Impact

There is a ~6.5% chance that finalizeInboundTransfer will not work.

## Proof of Concept

l1Address is L1GraphTokenGateway, suppose its address is 0xF000000000000000000000000000000000000000.

Then 0xF000000000000000000000000000000000000000 + 0x1111000000000000000000000000000000001111 > UINT160_MAX , meaning overflow.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Wrap the calculation in an unchecked block, which will make it behave correctly.