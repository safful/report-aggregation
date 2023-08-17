## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [fund lose or griefing in all order matching functions [matchOneToOneOrders(), matchOneToManyOrders(), matchOrders(), takeMultipleOneOrders(), takeOrders()] because condition (seller != buyer ) is not checked in any of them](https://github.com/code-423n4/2022-06-infinity-findings/issues/130) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L125-L364


# Vulnerability details

## Impact
Functions `matchOneToOneOrders()`, `matchOneToManyOrders()`, `matchOrders()`, `takeMultipleOneOrders()`, `takeOrders()` are for order matching and order execution and they validate different things about orders but there is no check for that `seller != buyer`, which can cause wrong order matching resulting in fund lose or fund theft or griefing. (it can be combined with other vulns to perform more damaging attacks)

## Proof of Concept
We only give proof of concept for `matchOneToManyOrders()` and other order execution/matching functions has similar bugs which root cause is not checking `seller != buyer`.
This is `matchOneToManyOrders()` code:
```
  /**
   @notice Matches one  order to many orders. Example: A buy order with 5 specific NFTs with 5 sell orders with those specific NFTs.
   @dev Can only be called by the match executor. Refunds gas cost incurred by the
        match executor to this contract. Checks whether the given complication can execute the match.
   @param makerOrder The one order to match
   @param manyMakerOrders Array of multiple orders to match the one order against
  */
  function matchOneToManyOrders(
    OrderTypes.MakerOrder calldata makerOrder,
    OrderTypes.MakerOrder[] calldata manyMakerOrders
  ) external {
    uint256 startGas = gasleft();
    require(msg.sender == MATCH_EXECUTOR, 'OME');
    require(_complications.contains(makerOrder.execParams[0]), 'invalid complication');
    require(
      IComplication(makerOrder.execParams[0]).canExecMatchOneToMany(makerOrder, manyMakerOrders),
      'cannot execute'
    );
    bytes32 makerOrderHash = _hash(makerOrder);
    require(isOrderValid(makerOrder, makerOrderHash), 'invalid maker order');
    uint256 ordersLength = manyMakerOrders.length;
    // the below 3 variables are copied to memory once to save on gas
    // an SLOAD costs minimum 100 gas where an MLOAD only costs minimum 3 gas
    // since these values won't change during function execution, we can save on gas by copying them to memory once
    // instead of SLOADing once for each loop iteration
    uint16 protocolFeeBps = PROTOCOL_FEE_BPS;
    uint32 wethTransferGasUnits = WETH_TRANSFER_GAS_UNITS;
    address weth = WETH;
    if (makerOrder.isSellOrder) {
      for (uint256 i = 0; i < ordersLength; ) {
        // 20000 for the SSTORE op that updates maker nonce status from zero to a non zero status
        uint256 startGasPerOrder = gasleft() + ((startGas + 20000 - gasleft()) / ordersLength);
        _matchOneMakerSellToManyMakerBuys(
          makerOrderHash,
          makerOrder,
          manyMakerOrders[i],
          startGasPerOrder,
          protocolFeeBps,
          wethTransferGasUnits,
          weth
        );
        unchecked {
          ++i;
        }
      }
      isUserOrderNonceExecutedOrCancelled[makerOrder.signer][makerOrder.constraints[5]] = true;
    } else {
      uint256 protocolFee;
      for (uint256 i = 0; i < ordersLength; ) {
        protocolFee += _matchOneMakerBuyToManyMakerSells(
          makerOrderHash,
          manyMakerOrders[i],
          makerOrder,
          protocolFeeBps
        );
        unchecked {
          ++i;
        }
      }
      isUserOrderNonceExecutedOrCancelled[makerOrder.signer][makerOrder.constraints[5]] = true;
      uint256 gasCost = (startGas - gasleft() + WETH_TRANSFER_GAS_UNITS) * tx.gasprice;
      // if the execution currency is weth, we can send the protocol fee and gas cost in one transfer to save gas
      // else we need to send the protocol fee separately in the execution currency
      // since the buyer is common across many sell orders, this part can be executed outside the above for loop
      // in contrast to the case where if the one order is a sell order, we need to do this in each for loop
      if (makerOrder.execParams[1] == weth) {
        IERC20(weth).safeTransferFrom(makerOrder.signer, address(this), protocolFee + gasCost);
      } else {
        IERC20(makerOrder.execParams[1]).safeTransferFrom(makerOrder.signer, address(this), protocolFee);
        IERC20(weth).safeTransferFrom(makerOrder.signer, address(this), gasCost);
      }
    }
  }
```
in its executions it calls `InfinityOrderBookComplication.canExecMatchOneToMany()`, `verifyMatchOneToManyOrders()`, `isOrderValid()` to see that if orders are valid and one order matched to all other orders but there is no check for `seller != buyer` in any of those functions. and also `ERC721` and `ERC20` allows funds and assets to be transferred from address to itself.
So it's possible for `matchOneToManyOrders()` to match one user sell orders to its buy orders which can cause fund theft or griefing. This is the scenario for fund lose in `matchOneToManyOrders()`:
1. let's assume orders `NFT` ids are for one collection for simplicity.
2. `NFT ID[1]` fair price is `8 ETH` and `NFT ID[2]` fair price is `2 ETH`.
3. `user1` wants to buy `NFT IDs[1,2]` at `10 ETH` (both of them) so he create one buy order and signs it.
4. `user1` wants to sell `NFT ID[1]` at `2.5 ETH` and sell `NFT ID[2]` at `8.5 ETH`. and he wants to sell them immediately after buying them so he create this two sell orders and sign them.
5. `attacker` who has `NFT ID[1]` creates an sell order for it at `7.5 ETH` and signs it.
6. off-chain machining engine sends this orders to `matchOneToManyOrders()`: many orders = [`(attacker sell ID[1] at 7.5 ETH)` , `(user1 sell ID[1] at 2.5 ETH)`] , one order = `(user1 buy IDs[1,2] at 10ETH)`
7. function `matchOneToManyOrders()` logic will check orders and their matching and all the checks would be passed for matching one order to many order(becase tokens lists intersects and numTokens are valids too (`1+1=2`))
8. function `matchOneToManyOrders()` would execute order and transfer funds and tokens which would result in: (transferring `7.5 ETH` from `user 1` to `attacker`) (transferring `2.5 ETH` from `user1` to `user1`) (transferring `NFT ID[1]` from `attacker` to `user1`) (transferring `NFT ID[1]` from `user1` to `user1`)
9. so in the end contract executed `user1` buy order `(user1 buy IDs[1,2] at 10ETH)` but `user` only received `NFT ID[1]` and didn't received `NFT ID[2]` so contract code perform operation contradiction to what `user1` has been signed.

Of course for this attack to work for `matchOneToManyOrders()` off-chain matching engine need to send wrong data but checks on the contract are not enough.

There are other scenarios for other functions that can cause griefing, for example for function `matchOrders()`:
a user can have multiple order to buy some tokens in list of ids. it's possible to match these old orders:
1. `user1` has this order: A:`(user1 BUY 1 of IDs[1,2,3])` and  B:`(user1 BUY 1 of IDs[1,4,5])` 
2. then the order B get executed for ID[1] and `user1` become the owner of `ID[1]`
3. `user1` wants to sell some of his tokens so he signs this order: C::`(user1 SELL 1 of IDs[1,6,7])`
4. matching engine would send order A and C with `constructedNfts=ID[1]` to `matchOrders()`.
5. `matchOrders()` would check conditions and would see that conditions are met and perform the transaction.
6. `user1` would pay some unnecessary order fee and it would become like griefing and DOS attack for him.

There may be other scenarios for this vuln to be harmful for users.

## Tools Used
VIM

## Recommended Mitigation Steps
add some checks to ensure that `seller != buyer`


