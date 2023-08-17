## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Reentrancy from matchOneToManyOrders](https://github.com/code-423n4/2022-06-infinity-findings/issues/184) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L178
https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L216
https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L230


# Vulnerability details

`matchOneToManyOrders` doesn't conform to Checks-Effects-Interactions pattern, and updates the maker order nonce only after the NFTs and payment have been sent.
Using this, a malicious user can re-enter the contract and re-fulfill the order using `takeOrders`.

## Impact
Orders can be executed twice. User funds would be lost.

## Proof of Concept
`matchOneToManyOrders` will set the order nonce as used only after the tokens are being sent:
```
  function matchOneToManyOrders(OrderTypes.MakerOrder calldata makerOrder, OrderTypes.MakerOrder[] calldata manyMakerOrders) external {
    ...
    if (makerOrder.isSellOrder) {
      for (uint256 i = 0; i < ordersLength; ) {
        ...
        _matchOneMakerSellToManyMakerBuys(...); // @audit will transfer tokens in here
        ...
      }
      //@audit setting nonce to be used only here
      isUserOrderNonceExecutedOrCancelled[makerOrder.signer][makerOrder.constraints[5]] = true;
    } else {
      for (uint256 i = 0; i < ordersLength; ) {
        protocolFee += _matchOneMakerBuyToManyMakerSells(...); // @audit will transfer tokens in here
        ...
      }
      //@audit setting nonce to be used only here
      isUserOrderNonceExecutedOrCancelled[makerOrder.signer][makerOrder.constraints[5]] = true;
      ...
  }
```

So we can see that tokens are being transferred before nonce is being set to executed.

Therefore, POC for an attack -
Alice wants to buy 2 unspecified WolfNFT, and she will pay via AMP, an ERC-777 token.
Malicious user Bob will set up an offer to sell 2 WolfNFT.
The MATCH_EXECUTOR will match the offers.
Bob will set up a contract such that upon receiving of AMP, it will call [`takeOrders`](https://github.com/code-423n4/2022-06-infinity/blob/main/contracts/core/InfinityExchange.sol#L336) with Alice's order, and 2 other WolfNFTs.
(Note that although `takeOrders` is `nonReentrant`, `matchOneToManyOrders` is not, and so the reentrancy will succeed.)

So in `takeOrders`, the contract will match Alice's order with Bob's NFTs, and then set Alice's order's nonce to true, then `matchOneToManyOrders` execution will resume, and again will set Alice's order's nonce to true.

Alice ended up buying 4 WolfNFTs although she only signed an order for 2. Tough luck, Alice.

(Note: a similar attack can be constructed via ERC721's onERC721Received.)

## Recommended Mitigation Steps
Conform to CEI and set the nonce to true before executing external calls.

