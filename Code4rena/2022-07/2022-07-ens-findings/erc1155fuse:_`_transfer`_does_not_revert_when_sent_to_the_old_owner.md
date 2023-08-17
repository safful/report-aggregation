## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [ERC1155Fuse: `_transfer` does not revert when sent to the old owner](https://github.com/code-423n4/2022-07-ens-findings/issues/179) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/ERC1155Fuse.sol#L274-L284


# Vulnerability details

## Impact

MED - the function of the protocol could be impacted

The `safeTransferFrom` does not comply with the ERC1155 standard when the token is sent to the old owner.

## Proof of Concept

According to the EIP-1155 standard for the `safeTransferFrom`:

> MUST revert if balance of holder for token `_id` is lower than the `_value` sent.


Let's say `alice` does not hold any token of `tokenId`, and `bob` holds one token of `tokenId`. Then alice tries to send one token of `tokenId` to bob with `safeTranferFrom(alice, bob, tokenId, 1, "")`.  In this case, even though alice's balance (= 0) is lower than the amount (= 1) sent, the `safeTransferFrom` will not revert. Thus, violating the EIP-1155 standard.
It can cause problems for other contracts using this token, since they assume the token was transferred if the `safeTransferFrom` does not revert. However, in the example above, no token was actually transferred.

```solidity
// https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/ERC1155Fuse.sol#L274-L284
// wrapper/ERC1155Fuse.sol::_transfer
// ERC1155Fuse::safeTransferFrom uses _transfer

274     function _transfer(
275         address from,
276         address to,
277         uint256 id,
278         uint256 amount,
279         bytes memory data
280     ) internal {
281         (address oldOwner, uint32 fuses, uint64 expiry) = getData(id);
282         if (oldOwner == to) {
283             return;
284         }
```

## Tools Used

none

## Recommended Mitigation Steps

Revert even if the `to` address already owns the token.



