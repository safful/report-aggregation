## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [hard-coded slippage may freeze user funds during market turbulence](https://github.com/code-423n4/2022-05-sturdy-findings/issues/133) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/main/smart-contracts/GeneralVault.sol#L125
https://github.com/code-423n4/2022-05-sturdy/blob/main/smart-contracts/LidoVault.sol#L130-L137


# Vulnerability details

## Impact
[GeneralVault.sol#L125](https://github.com/code-423n4/2022-05-sturdy/blob/main/smart-contracts/GeneralVault.sol#L125)
GeneralVault set a hardcoded slippage control of 99%. However, the underlying yield tokens price may go down.
If Luna/UST things happen again, users' funds may get locked.

[LidoVault.sol#L130-L137](https://github.com/code-423n4/2022-05-sturdy/blob/main/smart-contracts/LidoVault.sol#L130-L137)
Moreover, the withdrawal of the lidoVault takes a swap from the curve pool. 1 stEth worth 0.98 ETH at the time of writing.
The vault can not withdraw at the current market.

Given that users' funds would be locked in the lidoVault, I consider this a high-risk issue.

## Proof of Concept
[1 stEth  = 0.98 Eth](https://twitter.com/hasufl/status/1524717773959700481/photo/1)

[LidoVault.sol#L130-L137](https://github.com/code-423n4/2022-05-sturdy/blob/main/smart-contracts/LidoVault.sol#L130-L137)


## Tools Used

## Recommended Mitigation Steps

There are different ways to set the slippage.


The first one is to let users determine the maximum slippage they're willing to take.
The protocol front-end should set the recommended value for them.
```solidity
  function withdrawCollateral(
    address _asset,
    uint256 _amount,
    address _to,
    uint256 _minReceiveAmount
  ) external virtual {
      // ...
    require(withdrawAmount >= _minReceiveAmount, Errors.VT_WITHDRAW_AMOUNT_MISMATCH);
  }
```


The second one is have a slippage control parameters that's set by the operator.

```solidity
    // Exchange stETH -> ETH via Curve
    uint256 receivedETHAmount = CurveswapAdapter.swapExactTokensForTokens(
      _addressesProvider,
      _addressesProvider.getAddress('STETH_ETH_POOL'),
      LIDO,
      ETH,
      yieldStETH,
      maxSlippage
    );
```

```solidity
    function setMaxSlippage(uint256 _slippage) external onlyOperator {
        maxSlippage = _slippage;

        //@audit This action usually emit an event.
        emit SetMaxSlippage(msg.sender, slippage);
    }
```

These are two common ways to deal with this issue. I prefer the first one.
The market may corrupt really fast before the operator takes action.
It's nothing fun watching the number go down while having no option.

