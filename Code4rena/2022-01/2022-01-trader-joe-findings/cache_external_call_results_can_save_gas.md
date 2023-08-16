## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache external call results can save gas](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/236) 

# Handle

WatchPug


# Vulnerability details

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, external call results should be cached if they are being used for more than one time.

For example:

`factory.getPair(wavaxAddress, tokenAddress)` and `factory.getPair(tokenAddress, wavaxAddress)` in `LaunchEvent#createPair()`

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L377-L435

```solidity
function createPair() external isStopped(false) atPhase(Phase.PhaseThree) {
    // ...
    require(
        factory.getPair(wavaxAddress, tokenAddress) == address(0) ||
            IJoePair(
                IJoeFactory(factory).getPair(wavaxAddress, tokenAddress)
            ).totalSupply() ==
            0,
        "LaunchEvent: liquid pair already exists"
    );
    // ...
    pair = IJoePair(factory.getPair(tokenAddress, wavaxAddress));
    // ...
}
```

note: `factory.getPair(a, b)` 与  `factory.getPair(b, a)` 相同, see 
[code at github](https://github.com/traderjoe-xyz/joe-core/blob/5c2ca96c3835e7f2660f2904a1224bb7c8f3b7a7/contracts/traderjoe/JoeFactory.sol#L41-L42)
or
[code at avascan](https://avascan.info/blockchain/c/address/0x9Ad6C38BE94206cA50bb0d90783181662f0Cfa10/contract#:~:text=getPair%5Btoken1%5D%5Btoken0%5D%20%3D%20pair%3B%20//%20populate%20mapping%20in%20the%20reverse%20direction)

```solidity
getPair[token0][token1] = pair;
getPair[token1][token0] = pair; // populate mapping in the reverse direction
```

`IJoeFactory(factory).getPair(_token, wavax)` in `RocketJoeFactory#createRJLaunchEvent()`

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeFactory.sol#L122-L128

```solidity
require(
    IJoeFactory(factory).getPair(_token, wavax) == address(0) ||
        IJoePair(IJoeFactory(factory).getPair(_token, wavax))
            .totalSupply() ==
        0,
    "RJFactory: liquid pair already exists"
);
```


`token.decimals()` in `LaunchEvent#createPair()`

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L395-L405

```solidity
if (
    floorPrice > (wavaxReserve * 10**token.decimals()) / tokenAllocated
) {
    tokenAllocated = (wavaxReserve * 10**token.decimals()) / floorPrice;
    // ...
}
```

