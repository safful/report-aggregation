## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use bytes32 rather than string/bytes](https://github.com/code-423n4/2021-09-swivel-findings/issues/21) 

# Handle

defsec


# Vulnerability details

## Impact

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size. On the MarketPlace.sol contract, string memory variable can be replaced with bytes32 array. That will save gas on the contract.


## Proof of Concept

1. Navigate to "https://github.com/Swivel-Finance/gost/blob/v2/test/marketplace/MarketPlace.sol" contract.
2. Investigate createMarket function. n and s variables can be replaced with bytes32 variable.

```
  function createMarket(
    address u,
    uint256 m,
    address c,
    string memory n,
    string memory s,
    uint8 d
  ) public onlyAdmin(admin) returns (bool) {
    require(swivel != address(0), 'swivel contract address not set');
    // TODO can we live with the factory pattern here both bytecode size wise and CREATE opcode cost wise?
    address zctAddr = address(new ZcToken(u, m, n, s, d));
    address vAddr = address(new VaultTracker(m, c, swivel));
    markets[u][m] = Market(c, zctAddr, vAddr);

    emit Create(u, m, c, zctAddr, vAddr);

    return true;
  }
```

## Tools Used

None

## Recommended Mitigation Steps

Consider to replace string variables with bytes32. That should be definitely cheaper. 

