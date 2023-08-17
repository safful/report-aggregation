## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Wrong items length assertion in basic order](https://github.com/code-423n4/2022-05-opensea-seaport-findings/issues/129) 

# Lines of code

https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/BasicOrderFulfiller.sol#L346-L349


# Vulnerability details

When fulfilling a basic order we need to assert that the parameter `totalOriginalAdditionalRecipients` is less or equal than the length of `additionalRecipients` written in calldata.
However in `_prepareBasicFulfillmentFromCalldata` this assertion is incorrect [(L346)](https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/BasicOrderFulfiller.sol#L346-L349):
```js
        // Ensure supplied consideration array length is not less than original.
        _assertConsiderationLengthIsNotLessThanOriginalConsiderationLength(
            parameters.additionalRecipients.length + 1,
            parameters.totalOriginalAdditionalRecipients
        );
```
The way the function it's written ([L75](https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/Assertions.sol#L75-L83)), it accepts also a length smaller than the original by 1 (basically there shouldn't be a `+ 1` in the first argument).

Interestingly enough, in the case `additionalRecipients.length < totalOriginalAdditionalRecipients`, the inline-assembly for-loop at [(L506)](https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/BasicOrderFulfiller.sol#L506) will read consideration items out-of-bounds.
This can be a vector of exploits, as illustrated below.

## Proof of Concept
Alice makes the following offer: a basic order, with two `considerationItem`s. The second item has the following data:
```js
consideration[1] = {
	itemType: ...,
	token: ...,
	identifierOrCriteria: ...,
	startAmount: X,
	endAmount: X,
	recipient: Y,
}
```
The only quantities we need to track are the amounts `X` and recipient `Y`.

When fulfilling the order normally, the fulfiller will spend `X` tokens sending them to `Y`. It's possible however to exploit the previous bug in a way that the fulfiller won't need to make this transfer.

To do this, the fulfiller needs to craft the following calldata:

| calldata pointer |   correct calldata   |   exploit calldata  |
|-----------------:|:--------------------:|:-------------------:|
|              ... |          ...         |         ...         |
|            0x204 |   1 (tot original)   |   1 (tot original)  |
|            0x224 |  0x240 (head addRec) | 0x240 (head addRec) |
|            0x244 |   0x2a0 (head sign)  |  0x260 (head sign)  |
|            0x264 |   1 (length addRec)  |  0 (length addRec)  |
|            0x284 |      X (amount)      |   X (length sign)   |
|            0x2a4 |     Y (recipient)    |    Y (sign body)    |
|            0x2c4 |  0x40 (length sign)  |   0x00 (sign body)  |
|            0x2e4 | [correct Alice sign] |         ...         |
|            0x304 | [correct Alice sign] |         ...         |
|                  |                      |                     |

Basically writing `additionalRecipients = []` and making the signature length = `X`, with `Y` being the first 32 bytes.
Of course this signature will be invalid; however it doesn't matter since the exploiter can call `validate` with the correct signature beforehand.

The transaction trace will look like this:
- the assertion `_assertConsiderationLengthIsNotLessThanOriginalConsiderationLength` passes;
- the `orderHash` calculated is the correct one, since the for-loop over original consideration items picks up calldata at pointers {0x284, 0x2a4} [(L513)](https://github.com/code-423n4/2022-05-opensea-seaport/blob/main/contracts/lib/BasicOrderFulfiller.sol#L513-L550);
- the order was already validated beforehand, so the signature isn't read;
- at the end, during the tokens transfers, only `offer` and `consideration[0]` are transferred, since the code looks at `additionalRecipients` which is empty.

Conclusion:

Every Order that is "basic" and has two or more consideration items can be fulfilled in a way to not trade the _last_ consideration item in the list. The fulfiller spends less then normally, and a recipient doesn't get his due.

There's also an extra requirement which is stricter: this last item's `startAmount` (= endAmount) needs to be smallish (< 1e6). This is because this number becomes the signature bytes length, and we need to fill the calldata with extra zeroes to complete it. Realistically then the exploit will work only if the item is a ERC20 will low decimals.

I've made a hardhat test that exemplifies the exploit. [(Link to gist)](https://gist.github.com/0xsanson/e87e5fe26665c6cecaef2d9c4b0d53f4).

## Recommended Mitigation Steps
Remove the `+1` at L347.

