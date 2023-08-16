## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Remove fallback function](https://github.com/code-423n4/2021-11-unlock-findings/issues/94) 

# Handle

HardlyDifficult


# Vulnerability details

## Impact
Unimplemented calls do not revert, this may cause unexpected behavior in wallets or other contracts.

## Proof of Concept
Locks are ERC721s, they also implement some ERC20 style calls such as `transfer`. If a wallet or another contract attempted to treat the contract as a ERC77, `send` would incorrectly appear to work but nothing happens under the hood. It would be better if this call reverted so that the user was aware the function is not supported before even broadcasting the transaction (Metamask will warn you if estimate gas fails).

This test currently fails (i.e. calling send does not revert).

```
  it("Should fail on unknown calls", async () => {
      const mock777 = await erc777.at(lock.address);
      await reverts(
        mock777.send(destination, 1, '0x', { from: singleKeyOwner })
      )
    })
```

## Tools Used
`yarn test`

## Recommended Mitigation Steps
Remove this line https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/contracts/PublicLock.sol#L72

Per the comments there is not a clear reason it's currently included. The test suite still passes when it is removed.

