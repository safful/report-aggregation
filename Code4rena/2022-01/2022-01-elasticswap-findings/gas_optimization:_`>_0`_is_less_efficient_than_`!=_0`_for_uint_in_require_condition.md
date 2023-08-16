## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization: `> 0` is less efficient than `!= 0` for uint in require condition](https://github.com/code-423n4/2022-01-elasticswap-findings/issues/161) 

# Handle

gzeon


# Vulnerability details

## Impact
`> 0` is less gas efficient than `!= 0` for uint in require condition when optimizer is enabled
Ref: https://twitter.com/GalloDaSballo/status/1485430908165443590

## Proof of Concept
https://github.com/code-423n4/2022-01-elasticswap/blob/d107a198c0d10fbe254d69ffe5be3e40894ff078/elasticswap/src/contracts/Exchange.sol#L176
```
        require(this.totalSupply() > 0, "Exchange: INSUFFICIENT_LIQUIDITY");
```


