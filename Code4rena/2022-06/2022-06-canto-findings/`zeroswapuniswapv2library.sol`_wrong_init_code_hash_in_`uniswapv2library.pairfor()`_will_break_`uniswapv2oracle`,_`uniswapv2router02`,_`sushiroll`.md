## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`zeroswap/UniswapV2Library.sol` Wrong init code hash in `UniswapV2Library.pairFor()` will break `UniswapV2Oracle`, `UniswapV2Router02`, `SushiRoll`](https://github.com/code-423n4/2022-06-canto-findings/issues/164) 

# Lines of code

https://github.com/Plex-Engineer/zeroswap/blob/03507a80322112f4f3c723fc68bed0f138702836/contracts/uniswapv2/libraries/UniswapV2Library.sol#L20-L28


# Vulnerability details

```solidity
function pairFor(address factory, address tokenA, address tokenB) internal pure returns (address pair) {
    (address token0, address token1) = sortTokens(tokenA, tokenB);
    pair = address(uint(keccak256(abi.encodePacked(
            hex'ff',
            factory,
            keccak256(abi.encodePacked(token0, token1)),
            hex'e18a34eb0e04b04f7a0ac29a6e80748dca96319b42c54d679cb821dca90c6303' // init code hash
        ))));
}
```

The `init code hash` in `UniswapV2Library.pairFor()` should be updated since the code of `UniswapV2Pair` has been changed. Otherwise, the `pair` address calculated will be wrong, most likely non-existing address.

There are many other functions and other contracts across the codebase, including  `UniswapV2Oracle`, `UniswapV2Router02`, and `SushiRoll`, that rely on the `UniswapV2Library.pairFor()` function for the address of the pair, with the `UniswapV2Library.pairFor()` returning a wrong and non-existing address, these functions and contracts will malfunction.

### Recommendation

Update the init code hash from `hex'e18a34eb0e04b04f7a0ac29a6e80748dca96319b42c54d679cb821dca90c6303'` to the value of `UniswapV2Factory.pairCodeHash()`.

