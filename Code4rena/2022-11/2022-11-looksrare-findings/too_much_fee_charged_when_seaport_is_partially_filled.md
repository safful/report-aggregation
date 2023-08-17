## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-02

# [Too much fee charged when Seaport is partially filled](https://github.com/code-423n4/2022-11-looksrare-findings/issues/71) 

# Lines of code

https://github.com/code-423n4/2022-11-looksrare/blob/e3b2c053f722b0ca2dce3a3eb06f64859b8b7a6f/contracts/proxies/SeaportProxy.sol#L136-L147


# Vulnerability details

## Impact
When a user fulfills an order using SeaportProxy, fees are charged in the _handleFees function based on orders.price.
```solidity
    function _handleFees(
        BasicOrder[] calldata orders,
        uint256 feeBp,
        address feeRecipient
    ) private {
        address lastOrderCurrency;
        uint256 fee;
        uint256 ordersLength = orders.length;

        for (uint256 i; i < ordersLength; ) {
            address currency = orders[i].currency;
            uint256 orderFee = (orders[i].price * feeBp) / 10000;
```
According to the Seaport documentation, Seaport allows partial fulfillment of orders, which results in too much fee being charged when an order is partially filled
https://docs.opensea.io/v2.0/reference/seaport-overview#partial-fills

Consider feeBp == 2%
The order on Seaport has a fill status of 0/100 and each item is worth 1 eth.
User A fulfills the order using LooksRareAggregator.execute and sends 102 ETH, where order.price == 100 ETH.
Since the other user fulfilled the order before User A, when User A fulfills the order, the order status is 99/100
Eventually User A buys an item for 1 ETH but pays a fee of 2 ETH.
## Proof of Concept
https://github.com/code-423n4/2022-11-looksrare/blob/e3b2c053f722b0ca2dce3a3eb06f64859b8b7a6f/contracts/proxies/SeaportProxy.sol#L136-L147

## Tools Used
None
## Recommended Mitigation Steps
Consider charging fees based on the user's actual filled price