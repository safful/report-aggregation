## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Fix event params for `KeyManagerChanged`](https://github.com/code-423n4/2021-11-unlock-findings/issues/128) 

# Handle

HardlyDifficult


# Vulnerability details

## Impact
KeyManagerChanged does not emit the new manager address as expected.
Additionally there's a small gas savings of 1.5k gas by not emitting the event twice in `grantKeys`.

## Proof of Concept
Per the event param names, this event should emit the new keyManager's address. That would allow an indexer such as subgraph to track the current manager for each token. However the event currently emits address(0):

https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/contracts/mixins/MixinKeys.sol#L229

Change that line to:
`emit KeyManagerChanged(_tokenId, _keyManager);`

Additionally this line may be removed: https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/contracts/mixins/MixinGrantKeys.sol#L48 as the call right before it to `_setKeyManagerOf` will emit the event already.

## Tools Used
`yarn test`

## Recommended Mitigation Steps
When testing this change only one test failed, and it was due to assuming the index of the event: https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/test/Lock/grantKeys.js#L80

It would be nice to be more robust like some other tests are, e.g. https://github.com/code-423n4/2021-11-unlock/blob/52f3f3d0524dda28aea327181c3479d85782007b/smart-contracts/test/Lock/grantKeys.js#L49 

Also add a test to confirm that the keyManager is emitting in the event.

Personally I like Waffle for testing events: https://ethereum-waffle.readthedocs.io/en/latest/getting-started.html?highlight=emits#writing-tests

```
 it('Transfer emits event', async () => {
    await expect(token.transfer(walletTo.address, 7))
      .to.emit(token, 'Transfer')
      .withArgs(wallet.address, walletTo.address, 7);
  });
```

