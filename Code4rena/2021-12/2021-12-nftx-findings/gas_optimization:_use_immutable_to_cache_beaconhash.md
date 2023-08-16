## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas Optimization: Use immutable to cache beaconhash](https://github.com/code-423n4/2021-12-nftx-findings/issues/187) 

# Handle

gzeon


# Vulnerability details

## Impact
Calculation of `xTokenAddr` can further save gas by caching the creation hash as a immutable state.

https://github.com/code-423n4/2021-12-nftx/blob/194073f750b7e2c9a886ece34b6382b4f1355f36/nftx-protocol-v2/contracts/solidity/NFTXInventoryStaking.sol#L136
```
address tokenAddr = Create2.computeAddress(salt, keccak256(type(Create2BeaconProxy).creationCode));
```

## Recommended Mitigation Steps
```
    bytes32 internal immutable beaconhash = keccak256(type(Create2BeaconProxy).creationCode);
    function xTokenAddr(address baseToken) public view virtual override returns (address) {
        bytes32 salt = keccak256(abi.encodePacked(baseToken));
        address tokenAddr = Create2.computeAddress(salt, beaconhash);
        return tokenAddr;
    }
```

