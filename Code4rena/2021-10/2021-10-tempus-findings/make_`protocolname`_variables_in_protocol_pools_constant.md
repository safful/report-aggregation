## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Make `protocolName` variables in protocol pools constant](https://github.com/code-423n4/2021-10-tempus-findings/issues/11) 

# Handle

pmerkleplant


# Vulnerability details

## Impact
The `protocolName` variables in the protocol-specific `TempusPool`s are set as
_immutable_ but could be set as _constant_.

See [Compound](https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/pools/CompoundTempusPool.sol#L19), 
[Aave](https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/pools/AaveTempusPool.sol#L18), 
[Lido](https://github.com/code-423n4/2021-10-tempus/blob/main/contracts/pools/LidoTempusPool.sol#L9).

