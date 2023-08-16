## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Approvals not cleared after key transfer](https://github.com/code-423n4/2021-11-unlock-findings/issues/160) 

# Handle

cmichel


# Vulnerability details

The locks implement three different approval types, see `onlyKeyManagerOrApproved` for an overview:
- key manager (map `keyManagerOf`)
- single-person approvals (map `approved`). Cleared by `_clearApproval` or `_setKeyManagerOf`
- operator approvals (map `managerToOperatorApproved`)

The `MixinTransfer.transferFrom` requires any of the three approval types in the `onlyKeyManagerOrApproved` modifier on the tokenId to authenticate transfers from `from`.

Notice that if the `to` address previously had a key but it expired only the `_setKeyManagerOf` call is performed, which does not clear `approved` if the key manager was already set to 0:

```solidity
function transferFrom(
  address _from,
  address _recipient,
  uint _tokenId
)
  public
  onlyIfAlive
  hasValidKey(_from)
  onlyKeyManagerOrApproved(_tokenId)
{
  // @audit this is skipped if user had a key that expired
  if (toKey.tokenId == 0) {
    toKey.tokenId = _tokenId;
    _recordOwner(_recipient, _tokenId);
    // Clear any previous approvals
    _clearApproval(_tokenId);
  }

  if (previousExpiration <= block.timestamp) {
    // The recipient did not have a key, or had a key but it expired. The new expiration is the sender's key expiration
    // An expired key is no longer a valid key, so the new tokenID is the sender's tokenID
    toKey.expirationTimestamp = fromKey.expirationTimestamp;
    toKey.tokenId = _tokenId;

    // Reset the key Manager to the key owner
    // @audit  doesn't clear approval if key manager already was 0
    _setKeyManagerOf(_tokenId, address(0));

    _recordOwner(_recipient, _tokenId);
  }
  // ...
}

// 
function _setKeyManagerOf(
  uint _tokenId,
  address _keyManager
) internal
{
  // @audit-ok only clears approved if key manager updated
  if(keyManagerOf[_tokenId] != _keyManager) {
    keyManagerOf[_tokenId] = _keyManager;
    _clearApproval(_tokenId);
    emit KeyManagerChanged(_tokenId, address(0));
  }
}
```

## Impact
It's possible to sell someone a key and then claim it back as the approvals are not always cleared.

## POC
- Attacker A has a valuable key (`tokenId = 42`) with an expiry date far in the future.
- A sets approvals for their second attacker controlled account A' by calling `MixinKeys.setApprovalForAll(A', true)`, which sets `managerToOperatorApproved[A][A'] = true`.
- A clears the key manager by setting it to zero, for example, by transferring it to a second account that does not have a key yet, this calls the above `_setKeyManagerOf(42, address(0));` in `transferFrom`
- A sets single-token approval to A' by calling `MixinKeys.approve(A', 42)`, setting `approved[42] = A'`.
- A sells the token to a victim V for a discount (compared to purchasing it from the Lock). The victim needs to have owned a key before which already expired. The `transferFrom(A, V, 42)` call sets the owner of token 42 to `V`, but does not clear the `approved[42] == A'` field as described above. (`_setKeyManagerOf(_tokenId, address(0));` is called but the key manager was already zero, which then does not clear approvals.)
- A' can claim back the token by calling `transferFrom(V, A', 42)` and the `onlyKeyManagerOrApproved(42)` modifier will pass as `approved[42] == A'` is still set.

## Recommended Mitigation Steps
The `_setKeyManagerOf` function should not handle clearing approvals of single-token approvals (`approved`) as these are two separate approval types.
The `transferFrom` function should always call `_clearApproval` in the `(previousExpiration <= block.timestamp)` case.


