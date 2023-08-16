## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache storage variable in the stack can save gas](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/129) 

# Handle

WatchPug


# Vulnerability details

Cache storage variable in the stack can save gas.

For instance:

https://github.com/sushiswap/trident/blob/master/contracts/pool/IndexPool.sol#L140-L155

`outRecord.reserve` is accessed 2 times. 

https://github.com/sushiswap/trident/blob/master/contracts/pool/IndexPool.sol#L158-L179
https://github.com/sushiswap/trident/blob/master/contracts/pool/IndexPool.sol#L182-L205

`inRecord.reserve` is accessed 3 times.

