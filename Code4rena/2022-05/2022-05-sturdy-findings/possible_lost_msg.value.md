## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Possible lost msg.value](https://github.com/code-423n4/2022-05-sturdy-findings/issues/62) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/GeneralVault.sol#L75-L89
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/LidoVault.sol#L79-L104
https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/ConvexCurveLPVault.sol#L131-L149


# Vulnerability details

## Impact
Possible lost value in `depositCollateral` function call

## Proof of Concept
In call `depositCollateral` can will send value and the asset can be an ERC20(!= address(0)), if `LidoVault` and `ConvexCurveLPVault` contract receive this call the fouds will lost
Also in **LidoVault, L88**, if send as asset ETH(== address(0)) and send more value than `_amount`(msg.value > _amount), the exedent will lost

## Recommended Mitigation Steps

In **GeneralVault**, `depositCollateral` function:
 - Check if the `msg.value` is zero when the `_asset` is ERC20(!= address(0))
 - Check if the `msg.value` is equeal to `_amount` when the `_asset` ETH(== address(0))

```solidity
function depositCollateral(address _asset, uint256 _amount) external payable virtual {
  if (_asset != address(0)) { // asset = ERC20
    require(msg.value == 0, <AN ERROR FROM Errors LIBRARY>);
  } else { // asset = ETH
    require(msg.value == _amount, <AN ERROR FROM Errors LIBRARY>);
  }

  // Deposit asset to vault and receive stAsset
  // Ex: if user deposit 100ETH, this will deposit 100ETH to Lido and receive 100stETH TODO No Lido
  (address _stAsset, uint256 _stAssetAmount) = _depositToYieldPool(_asset, _amount);

  // Deposit stAsset to lendingPool, then user will get aToken of stAsset
  ILendingPool(_addressesProvider.getLendingPool()).deposit(
    _stAsset,
    _stAssetAmount,
    msg.sender,
    0
  );

  emit DepositCollateral(_asset, msg.sender, _amount);
}
```
Also can remove the `require(msg.value > 0, Errors.VT_COLLATERAL_DEPOSIT_REQUIRE_ETH);` in **LidoVault, L88**


