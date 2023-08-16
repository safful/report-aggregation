## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [transfer() may break in future ETH upgrade](https://github.com/code-423n4/2022-01-openleverage-findings/issues/228) 

# Handle

gzeon


# Vulnerability details

## Impact
`transfer()` only forward 2300 gas which may break when gas cost change in a future ETH upgrade
see: https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

## Proof of Concept
https://github.com/code-423n4/2022-01-openleverage/blob/501e8f5c7ebaf1242572712626a77a3d65bdd3ad/openleverage-contracts/contracts/OpenLevV1Lib.sol#L253
```
            payable(to).transfer(amount);
```

## Recommended Mitigation Steps
use call() instead

