## Tags

- bug
- 2 (Med Risk)
- high quality report
- primary issue
- selected for report
- sponsor confirmed
- M-07

# [Liquidators can be tricked to operate with LiquidationPairs that were deployed using the LiquidationPairFactory but they configured the LiquidationSource as a fake malicious contract](https://github.com/code-423n4/2023-08-pooltogether-findings/issues/68) 

# Lines of code

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L65-L108


# Vulnerability details

## Impact
- Users and Bots can be tricked into operating with LiquidationPairs that were deployed using the LiquidationPairFactory but they configured the LiquidationSource as a fake malicious contract that will allow the Liquidator's creator to steal all the POOL tokens that were meant to be used to liquidate the Vault's Yield

## Proof of Concept
- The current implementation of the [`LiquidationPairFactory::createPair()`](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L65-L108) allows the callers to set the LiquidationSource as any arbitrary address with no restrictions.
- The problem is that the address of the `_source` parameter may not necessarily be a real vault contract, and even though the `_source` address is set as a fake malicious contract, the LiquidationPair will be created and added to the `deployedPairs` mapping, and [as stated in the protocol's documentation, **any LiquidationPair created by the factory determines if a pair is legitimate or not**](https://dev.pooltogether.com/protocol/next/guides/liquidating-yield#find-the-liquidation-pair)

![Liquidation Pair Protocol's Documentation](https://res.cloudinary.com/djt3zbrr3/image/upload/v1691397160/PoolTogether/LiquidationPairFactory_Documentation.png)

- So, if a LiquidationPair is created by the LiquidationPairFactory may allow malicious users to trick users who want to liquidate the Vault's Yield to operate with a LiquidationPair who'll end up stealing their POOL tokens (tokenIn) when swapping tokens.

- Practical Example of how the LiquidationPair would steal the user's assets, (Keep in mind that `source` is not the address of a Vault, but an arbitrary contract)
  - The fake `source` contract would look like this:
```solidity
contract FakeSource {

  function targetOf(address _token) external view returns (address) {
    return <ContractCreatorAddress>;
  }

  function liquidate(
    address _account,
    address _tokenIn,
    uint256 _amountIn,
    address _tokenOut,
    uint256 _amountOut
  ) public virtual override returns (bool) {
    return true;
  }

}

```
1. Calling [`LiquidationPair.target()`](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L71) will return `source.targetOf(tokenIn)`, and the `source` contract can return any address when the `targetOf()` is called, so, let's say that will return the address of its creator.
  - so, `LiquidationPair.target()` will return the address of its creator, instead of returning the expected address of the `PrizePool` contract

2. Calling [`LiquidationPair::swapExactAmount()`](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L75) will do some computations prior to call `source.liquidate()`, and the `source.liquidate()` can just return true not to cause the tx to be reverted.
  - So, `LiquidationPair::swapExactAmount()` will basically do nothing.


- With the above points in mind, let's see what would be the result of swapping using the LiquidationRouter contract
```solidity
function swapExactAmountOut(
  LiquidationPair _liquidationPair,
  address _receiver,
  uint256 _amountOut,
  uint256 _amountInMax
) external onlyTrustedLiquidationPair(_liquidationPair) returns (uint256) {

  //@audit-issue => The `tokenIn` will be transferred to the address of the LiquidationPair's creator instead of the PrizePool contract (Point 1) <====== Point 1 ======>
  IERC20(_liquidationPair.tokenIn()).safeTransferFrom(
    msg.sender,
    _liquidationPair.target(),
    _liquidationPair.computeExactAmountIn(_amountOut)
  );

  //@audit-issue => This call will basically do nothing, just return a true not to cause the tx to be reverted (Point 2)  <====== Point 2 ======>
  uint256 amountIn = _liquidationPair.swapExactAmountOut(_receiver, _amountOut, _amountInMax);

  emit SwappedExactAmountOut(_liquidationPair, _receiver, _amountOut, _amountInMax, amountIn);

  return amountIn;
}
```

## Tools Used
Manual Audit

## Recommended Mitigation Steps
- Use the `deployedVaults` mapping of the `VaultFactory` contract to validate if the inputted address of the `_source` parameter is a valid vault supported by the Protocol.
  - Additionally, it could be a good idea to set the `_tokenIn` and `_tokenOut` by pulling the values that are already set up in the vault.

```solidity
function createPair(
  ILiquidationSource _source,
- address _tokenIn,
- address _tokenOut,
  uint32 _periodLength,
  uint32 _periodOffset,
  uint32 _targetFirstSaleTime,
  SD59x18 _decayConstant,
  uint112 _initialAmountIn,
  uint112 _initialAmountOut,
  uint256 _minimumAuctionAmount
) external returns (LiquidationPair) {

+ require(VaultFactory.deployedVaults(address(_source)) == true, "_source address is not a supported Vault");
+ address _prizePool = _source.prizePool();
+ address _tokenIn = _prizePool.prizeToken();
+ address _tokenOut = address(_source);
  
  LiquidationPair _liquidationPair = new LiquidationPair(
    _source,
    _tokenIn,
    _tokenOut,
    _periodLength,
    _periodOffset,
    _targetFirstSaleTime,
    _decayConstant,
    _initialAmountIn,
    _initialAmountOut,
    _minimumAuctionAmount
  );

  allPairs.push(_liquidationPair);
  deployedPairs[_liquidationPair] = true;

  emit PairCreated(
    _liquidationPair,
    _source,
    _tokenIn,
    _tokenOut,
    _periodLength,
    _periodOffset,
    _targetFirstSaleTime,
    _decayConstant,
    _initialAmountIn,
    _initialAmountOut,
    _minimumAuctionAmount
  );

  return _liquidationPair;
}

```


## Assessed type

Invalid Validation