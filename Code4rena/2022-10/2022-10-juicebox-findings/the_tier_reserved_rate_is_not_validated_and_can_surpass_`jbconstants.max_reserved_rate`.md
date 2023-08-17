## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- satisfactory
- selected for report
- M-08

# [The tier reserved rate is not validated and can surpass `JBConstants.MAX_RESERVED_RATE`](https://github.com/code-423n4/2022-10-juicebox-findings/issues/201) 

# Lines of code

https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L1224-L1259
https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L566
https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/JB721Delegate.sol#L142


# Vulnerability details

If the reserved rate of a tier is set to a value > `JBConstants.MAX_RESERVED_RATE`, the `JBTiered721DelegateStore._numberOfReservedTokensOutstandingFor` function will return way more outstanding reserved tokens (up to ~6 times more than allowed - **2^16 - 1** due to the manual cast of `reservedRate` to `uint16` divided by `JBConstants.MAX_RESERVED_RATE = 10_000`). This inflated value is used in the `JBTiered721DelegateStore.totalRedemptionWeight` function to calculate the cumulative redemption weight of all tokens across all tiers.

This higher-than-expected redemption weight will lower the `reclaimAmount` calculated in the `JB721Delegate.redeemParams` function. Depending on the values of `_data.overflow` and `_redemptionWeight`, the calculated `reclaimAmount` can be **0** (due to rounding down, [see here](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/JB721Delegate.sol#L142)) or a smaller than anticipated value, leading to burned NFT tokens from the user and no redemptions.

## Impact

The owner of an NFT contract can add tiers with higher than usual reserved rates (and mint an appropriate number of NFTs to bypass all conditions in the `JBTiered721DelegateStore._numberOfReservedTokensOutstandingFor`), which will lead to a lower-than-expected redemption amount for users.

## Proof of Concept

[JBTiered721DelegateStore.\_numberOfReservedTokensOutstandingFor](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L1224-L1259)

```solidity
function _numberOfReservedTokensOutstandingFor(
  address _nft,
  uint256 _tierId,
  JBStored721Tier memory _storedTier
) internal view returns (uint256) {
  // Invalid tier or no reserved rate?
  if (_storedTier.initialQuantity == 0 || _storedTier.reservedRate == 0) return 0;

  // No token minted yet? Round up to 1.
  if (_storedTier.initialQuantity == _storedTier.remainingQuantity) return 1;

  // The number of reserved tokens of the tier already minted.
  uint256 _reserveTokensMinted = numberOfReservesMintedFor[_nft][_tierId];

  // If only the reserved token (from the rounding up) has been minted so far, return 0.
  if (_storedTier.initialQuantity - _reserveTokensMinted == _storedTier.remainingQuantity)
    return 0;

  // Get a reference to the number of tokens already minted in the tier, not counting reserves or burned tokens.
  uint256 _numberOfNonReservesMinted = _storedTier.initialQuantity -
    _storedTier.remainingQuantity -
    _reserveTokensMinted;

  // Store the numerator common to the next two calculations.
  uint256 _numerator = uint256(_numberOfNonReservesMinted * _storedTier.reservedRate);

  // Get the number of reserved tokens mintable given the number of non reserved tokens minted. This will round down.
  uint256 _numberReservedTokensMintable = _numerator / JBConstants.MAX_RESERVED_RATE;

  // Round up.
  if (_numerator - JBConstants.MAX_RESERVED_RATE * _numberReservedTokensMintable > 0)
    ++_numberReservedTokensMintable;

  // Return the difference between the amount mintable and the amount already minted.
  return _numberReservedTokensMintable - _reserveTokensMinted;
}
```

[JBTiered721DelegateStore.totalRedemptionWeight](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L566)

The `JBTiered721DelegateStore._numberOfReservedTokensOutstandingFor` function is called from within the `JBTiered721DelegateStore.totalRedemptionWeight` function. This allows for inflating the total redemption weight.

```solidity
function totalRedemptionWeight(address _nft) public view override returns (uint256 weight) {
  // Keep a reference to the greatest tier ID.
  uint256 _maxTierId = maxTierIdOf[_nft];

  // Keep a reference to the tier being iterated on.
  JBStored721Tier memory _storedTier;

  // Add each token's tier's contribution floor to the weight.
  for (uint256 _i; _i < _maxTierId; ) {
    // Keep a reference to the stored tier.
    _storedTier = _storedTierOf[_nft][_i + 1];

    // Add the tier's contribution floor multiplied by the quantity minted.
    weight +=
      (_storedTier.contributionFloor *
        (_storedTier.initialQuantity - _storedTier.remainingQuantity)) +
      _numberOfReservedTokensOutstandingFor(_nft, _i, _storedTier);

    unchecked {
      ++_i;
    }
  }
}
```

[JBTiered721Delegate.\_totalRedemptionWeight](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721Delegate.sol#L712)

`JBTiered721DelegateStore.totalRedemptionWeight` is called in the `JBTiered721Delegate._totalRedemptionWeight` function.

```solidity
function _totalRedemptionWeight() internal view virtual override returns (uint256) {
  return store.totalRedemptionWeight(address(this));
}
```

[abstract/JB721Delegate.redeemParams](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/JB721Delegate.sol#L139)

This `JBTiered721Delegate._totalRedemptionWeight` function is then called in the `JB721Delegate.redeemParams` function, which ultimately calculates the `reclaimAmount` given an overflow and `_decodedTokenIds`.

`uint256 _base = PRBMath.mulDiv(_data.overflow, _redemptionWeight, _total);` in [line 142](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/JB721Delegate.sol#L142) will lead to a lower `_base` due to the inflated denumerator `_total`.

```solidity
function redeemParams(JBRedeemParamsData calldata _data)
  external
  view
  override
  returns (
    uint256 reclaimAmount,
    string memory memo,
    JBRedemptionDelegateAllocation[] memory delegateAllocations
  )
{
  // Make sure fungible project tokens aren't being redeemed too.
  if (_data.tokenCount > 0) revert UNEXPECTED_TOKEN_REDEEMED();

  // Check the 4 bytes interfaceId and handle the case where the metadata was not intended for this contract
  if (
    _data.metadata.length < 4 || bytes4(_data.metadata[0:4]) != type(IJB721Delegate).interfaceId
  ) {
    revert INVALID_REDEMPTION_METADATA();
  }

  // Set the only delegate allocation to be a callback to this contract.
  delegateAllocations = new JBRedemptionDelegateAllocation[](1);
  delegateAllocations[0] = JBRedemptionDelegateAllocation(this, 0);

  // If redemption rate is 0, nothing can be reclaimed from the treasury
  if (_data.redemptionRate == 0) return (0, _data.memo, delegateAllocations);

  // Decode the metadata
  (, uint256[] memory _decodedTokenIds) = abi.decode(_data.metadata, (bytes4, uint256[]));

  // Get a reference to the redemption rate of the provided tokens.
  uint256 _redemptionWeight = _redemptionWeightOf(_decodedTokenIds);

  // Get a reference to the total redemption weight.
  uint256 _total = _totalRedemptionWeight(); // @audit-info Uses the inflated total redemption weight

  // Get a reference to the linear proportion.
  uint256 _base = PRBMath.mulDiv(_data.overflow, _redemptionWeight, _total);

  // These conditions are all part of the same curve. Edge conditions are separated because fewer operation are necessary.
  if (_data.redemptionRate == JBConstants.MAX_REDEMPTION_RATE)
    return (_base, _data.memo, delegateAllocations);

  // Return the weighted overflow, and this contract as the delegate so that tokens can be deleted.
  return (
    PRBMath.mulDiv(
      _base,
      _data.redemptionRate +
        PRBMath.mulDiv(
          _redemptionWeight,
          JBConstants.MAX_REDEMPTION_RATE - _data.redemptionRate,
          _total
        ),
      JBConstants.MAX_REDEMPTION_RATE
    ),
    _data.memo,
    delegateAllocations
  );
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Consider validating the tier reserved rate `reservedRate` in the `JBTiered721DelegateStore.recordAddTiers` function to ensure the reserved rate is not greater than `JBConstants.MAX_RESERVED_RATE`.
