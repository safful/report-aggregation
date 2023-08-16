## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [matchOrders could/should check market](https://github.com/code-423n4/2021-06-tracer-findings/issues/6) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function matchOrders of TracerPerpetualSwaps.sol doesn't check that the contract itself is indeed equal to order1.market and order2.market.
The function executeTrade Trader.sol, which calls the matchOrders, can deal with multiple markets.
Suppose there would be a mistake in executeTrade,  or in a future version, the matchOrders would be done in the wrong market.

## Proof of Concept
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/TracerPerpetualSwaps.sol#L216
function matchOrders( Perpetuals.Order memory order1, Perpetuals.Order memory order2, uint256 fillAmount )


// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Trader.sol#L67
 function executeTrade(Types.SignedLimitOrder[] memory makers, Types.SignedLimitOrder[] memory takers) external  override {
...
 (bool success, ) = makeOrder.market.call(
                abi.encodePacked(
                    ITracerPerpetualSwaps(makeOrder.market).matchOrders.selector,
                    abi.encode(makeOrder, takeOrder, fillAmount)
                )
            );


// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/lib/LibPerpetuals.sol#L128
function canMatch( Order memory a, uint256 aFilled,Order memory b, uint256 bFilled ) internal view returns (bool) {
    ...
        bool marketsMatch = a.market == b.market;

## Tools Used

## Recommended Mitigation Steps
Add something like:
require ( order1.market == address(this), "Wrong market");

Note: canMatch already verifies that  order1.market== order2.market


