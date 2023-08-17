## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Create a short call order with non empty floor makes the option impossible to exercise and withdraw](https://github.com/code-423n4/2022-06-putty-findings/issues/369) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L296-L298


# Vulnerability details

## Impact

**HIGH** - assets can be lost
If a short call order is created with non empty floorTokens array, the taker cannot exercise. Also, the maker cannot withdraw after the expiration. The maker will still get premium when the order is filled. If the non empty floorTokens array was included as an accident, it is a loss for both parties: the taker loses premium without possible exercise, the maker loses the locked ERC20s and ERC721s.
This bug is not suitable for exploitation to get a 'free' premium by creating not exercisable options, because the maker will lose the ERC20s and ERC721s without getting any strike. In that sense it is similar but different issue to the `Create a short put order with zero tokenAmount makes the option impossible to exercise`, therefore reported separately.


## Proof of Concept

- [proof of concept](https://gist.github.com/zzzitron/9f83516255fa6153a4deb04f2163a0b3#file-2022-07-puttyv2-t-sol-L153-L202)
- [reference case](https://gist.github.com/zzzitron/9f83516255fa6153a4deb04f2163a0b3#file-2022-07-puttyv2-t-sol-L194-L21://gist.github.com/zzzitron/9f83516255fa6153a4deb04f2163a0b3#file-2022-07-puttyv2-t-sol-L204-L226)

The proof of concept shows a scenario where babe makes an short call order with non empty `floorTokens` array. Bob filled the order, and now he has long call option NFT. He wants to exercise his option and calls `exercise`. There are two cases.
- case 1: he calls exercise with empty `floorAssetTokenIds` array
- case 2: he calls exercise with non-empty `floorAssetTokenIds` array with matching length to the `orders.floorTokens`

In the case1, [the input `floorAssetTokenIds` were checked to be empty for put orders](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L406), and his call passes this requirement. But eventually `_transferFloorsIn` was called and he gets `Index out of bounds` error, because `floorTokens` is not empty [which does not match with empty `floorAssetTokenIds`](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L627-L629).
```solidity
// case 1
  // PuttyV2.sol: _transferFloorsIn called by exercise
  // The floorTokens and floorTokenIds do not match the lenghts
  // floorTokens.length is not zero, while floorTokenIds.length is zero

       ERC721(floorTokens[i]).safeTransferFrom(from, address(this), floorTokenIds[i]);
```

In the case2, [the input `floorAssetTokenIds` were checked to be empty for put orders](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L406), but it is not empty. So it reverts.
```
// case2
// PuttyV2.sol: exercise
// non empty floorAssetTokenIds array is passed for put option, it will revert

        !order.isCall
            ? require(floorAssetTokenIds.length == order.floorTokens.length, "Wrong amount of floor tokenIds")
            : require(floorAssetTokenIds.length == 0, "Invalid floor tokenIds length");
```

After the option is expired, the maker - babe is trying to withdraw but fails due to the [same issue with the case1](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L516).
```solidity
// maker trying to withdraw
// PuttyV2.sol: withdraw

  _transferFloorsOut(order.floorTokens, positionFloorAssetTokenIds[floorPositionId]);
```

Note on the poc:
- The [test for case1 is commented out](https://gist.github.com/zzzitron/9f83516255fa6153a4deb04f2163a0b3#file-2022-07-puttyv2-t-sol-L182-L183) because foundry could not catch the revert. But by running the test with un-commenting these lines will show that the call reverts with `Index out of bounds`.
- For the same reason the [withdraw](https://gist.github.com/zzzitron/9f83516255fa6153a4deb04f2163a0b3#file-2022-07-puttyv2-t-sol-L199-L200) also is commented out
- The reference case just shows that it works as intended when the order does not contain non-empty `floorTokens`.

## Tools Used

foundry

## Recommended Mitigation Steps

It happens because the [`fillOrder` does not ensure](https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L296-L298) the `order.floorTokens` to be empty when the order is short call.



