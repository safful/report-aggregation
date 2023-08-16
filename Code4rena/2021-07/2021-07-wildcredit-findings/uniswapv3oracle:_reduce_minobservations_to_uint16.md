## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [UniswapV3Oracle: Reduce minObservations to uint16](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/55) 

# Handle

greiart


# Vulnerability details

## Impact

Will help prevent erraneous `minObservations` values from being set (ie. `> 65535`) by the owner without needing checks. Otherwise, the `isPoolValid` will always return false, causing reverts in calling `tokenPrice` and `addPool` functions (and other functions calling these).

## Referenced Codelines

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/UniswapV3Oracle.sol#L25](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/UniswapV3Oracle.sol#L25)

[https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/UniswapV3Oracle.sol#L101](https://github.com/code-423n4/2021-07-wildcredit/blob/main/contracts/UniswapV3Oracle.sol#L101)

## Proof Of Concept

The maximum number of observations available is `65535` (see [https://github.com/Uniswap/uniswap-v3-core/blob/main/contracts/UniswapV3Pool.sol#L39](https://github.com/Uniswap/uniswap-v3-core/blob/main/contracts/UniswapV3Pool.sol#L39)), which is equivalent to `type(uint16).max`.

Hence, 

- `uint public minObservations` can be reduced to `uint16 public minObservations`.
- `(, , , , uint observationSlots , ,) = IUniswapV3Pool(poolAddress).slot0();` becomes `(, , , , uint16 observationSlots , ,) = IUniswapV3Pool(poolAddress).slot0();`

