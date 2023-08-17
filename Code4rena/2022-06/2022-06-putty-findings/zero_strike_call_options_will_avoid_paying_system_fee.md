## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- old-submission-method

# [Zero strike call options will avoid paying system fee](https://github.com/code-423n4/2022-06-putty-findings/issues/373) 

# Lines of code

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L494-L506


# Vulnerability details

Zero and near zero strike calls are common derivative type. For such derivatives the system will not be receiving fees are the fee is now formulated as a fraction of order strike.

Also, it can be a problem for OTM call options, when the option itself is nearly worthless, while the fee will be substantial as strike will be big. Say 1k ETH BAYC call doesn't have much value, but the associated fee will be 10x of usual fee, i.e. substantial, while there is nothing to justify that.

Marking this as medium severity as that's a design specifics that can turn off or distort core system fee gathering.

## Proof of Concept

Currently fee is linked to the order strike which makes it vary heavily for different types of orders, for example deep ITM and OTM calls:

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L494-L506

```solidity
        // transfer strike to owner if put is expired or call is exercised
        if ((order.isCall && isExercised) || (!order.isCall && !isExercised)) {
            // send the fee to the admin/DAO if fee is greater than 0%
            uint256 feeAmount = 0;
            if (fee > 0) {
                feeAmount = (order.strike * fee) / 1000;
                ERC20(order.baseAsset).safeTransfer(owner(), feeAmount);
            }

            ERC20(order.baseAsset).safeTransfer(msg.sender, order.strike - feeAmount);

            return;
        }
```

## Recommended Mitigation Steps

Consider linking the fee to option premium as this is option value that cannot be easily manipulated and exactly corresponds to the trading volume of the system.

I.e. consider moving fee gathering to fillOrder:

https://github.com/code-423n4/2022-06-putty/blob/3b6b844bc39e897bd0bbb69897f2deff12dc3893/contracts/src/PuttyV2.sol#L322-L340

```solidity
        // transfer premium to whoever is short from whomever is long
        if (order.isLong) {
            ERC20(order.baseAsset).safeTransferFrom(order.maker, msg.sender, order.premium);
        } else {
            // handle the case where the user uses native ETH instead of WETH to pay the premium
            if (weth == order.baseAsset && msg.value > 0) {
                // check enough ETH was sent to cover the premium
                require(msg.value == order.premium, "Incorrect ETH amount sent");

                // convert ETH to WETH and send premium to maker
                // converting to WETH instead of forwarding native ETH to the maker has two benefits;
                // 1) active market makers will mostly be using WETH not native ETH
                // 2) attack surface for re-entrancy is reduced
                IWETH(weth).deposit{value: msg.value}();
                IWETH(weth).transfer(order.maker, msg.value);
            } else {
                ERC20(order.baseAsset).safeTransferFrom(msg.sender, order.maker, order.premium);
            }
        }
```

