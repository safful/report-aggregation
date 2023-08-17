## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [User fund loss in supplyTokenTo() because of rounding](https://github.com/code-423n4/2022-04-pooltogether-findings/issues/37) 

# Lines of code

https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L231-L242
https://github.com/pooltogether/aave-v3-yield-source/blob/e63d1b0e396a5bce89f093630c282ca1c6627e44/contracts/AaveV3YieldSource.sol#L357-L362


# Vulnerability details

## Impact
When user use `supplyTokenTo()` to deposit his tokens and get `share` in `FeildSource` because of rounding in division user gets lower amount of `share`. for example if token's `_decimal` was `1` and `totalSupply()` was `1000` and `aToken.balanceOf(FieldSource.address)` was `2100` (becasue of profits in `Aave Pool` `balance` is higher than `supply`), then if user deposit `4` token to the contract with `supplyTokenTo()`, contract is going to `mint` only `1` share for that user and if user calls `YeildToken.balanceOf(user)` the return value is going to be `2` and user already lost half of his deposit.
Of course if `_precision ` was high this loss is going to be low enough to ignore but in case of low `_precision` and high price `token` and high `balance / supply` ratio this loss is going to be noticeable.

## Proof of Concept
This is the code of `supplyTokenTo()`:
```
  function supplyTokenTo(uint256 _depositAmount, address _to) external override nonReentrant {
    uint256 _shares = _tokenToShares(_depositAmount);
    require(_shares > 0, "AaveV3YS/shares-gt-zero");

    address _underlyingAssetAddress = _tokenAddress();
    IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender, address(this), _depositAmount);
    _pool().supply(_underlyingAssetAddress, _depositAmount, address(this), REFERRAL_CODE);

    _mint(_to, _shares);

    emit SuppliedTokenTo(msg.sender, _shares, _depositAmount, _to);
  }
```
which in line: `_shares = _tokenToShares(_depositAmount)` trying to calculated `shares` corresponding to the number of tokens supplied. and then transfer `_depositAmount` from user and `mint` shares amount for user.
the problem is that if user convert `_shares` to token, he is going to receive lower amount because in most cases:
```
_depositAmount > _sharesToToken(_tokenToShares(_depositAmount))
```
and that's because of rounding in division. Value of `_shares` is less than _depositAmount. so `YeildSource` should only take part of `_depositAmount` that equals to `_sharesToToken(_tokenToShares(_depositAmount))` and mint `_share` for user.

Of course if `_precision` was high and `aToken.balanceOf(FieldSource.address) / totalSupply()` was low, then this amount will be insignificant, but for some cases it can be harmful for users. for example this conditions:
- `_perecision` is low like 1 or 2.
- `token` value is very high like BTC.
- `aToken.balanceOf(FieldSource.address) / totalSupply()` is high due to manipulation or profit in `Aave pool`.

## Tools Used
VIM

## Recommended Mitigation Steps
To resolve this issue this can be done:
```
  function supplyTokenTo(uint256 _depositAmount, address _to) external override nonReentrant {
    uint256 _shares = _tokenToShares(_depositAmount);
    require(_shares > 0, "AaveV3YS/shares-gt-zero");
    
    _depositAmount = _sharesToToken(_shares); // added hero to only take correct amount of user tokens
    address _underlyingAssetAddress = _tokenAddress();
    IERC20(_underlyingAssetAddress).safeTransferFrom(msg.sender, address(this), _depositAmount);
    _pool().supply(_underlyingAssetAddress, _depositAmount, address(this), REFERRAL_CODE);

    _mint(_to, _shares);

    emit SuppliedTokenTo(msg.sender, _shares, _depositAmount, _to);
  }
```

