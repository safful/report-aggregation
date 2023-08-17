## Tags

- 3 (High Risk)
- satisfactory
- selected for report
- sponsor confirmed
- MR-H-02

# [Mitigation of H-02: Issue not fully mitigated](https://github.com/code-423n4/2023-02-reserve-mitigation-contest-findings/issues/49) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/610cfca553beea41b9508abbfbf4ee4ce16cbc12/contracts/p1/mixins/RecollateralizationLib.sol#L146-L245


# Vulnerability details

# Mitigation of H-02: Issue not fully mitigated
### Original issue: [H-02: Basket range formula is inefficient, leading the protocol to unnecessary haircut](https://github.com/code-423n4/2023-01-reserve-findings/issues/235)


## Not mitigated - top range can still be too high, leading to unnecessary haircut
* The applied mitigation follows the line of the mitigation suggested (disclosure: by me :)) in the original issue, however after reviewing it I found out that it doesn't fully mitigate the issue.
* The original issue was that basket range band is too wide, with both top range being too high and bottom range too low
* The bottom range is mitigated now
* As for the top range - even though it's more efficient now, it still can result in a top range that doesn't make sense.

## Impact
Protocol might go for an unnecessary haircut, causing a loss for RToken holders.
In the scenario below we can trade to get ~99% of baskets needed, but instead the protocol goes for a 50% haircut.

After the haircut the baskets held per supply ratio might grow back via `handoutExcessAssets` and `Furnace` however:
* Not all excess asset goes to `Furnace`
* `Furnace` grows slowly over time and in the meantime
    * Redemption would be at the lower baskets per supply
    * New users can issue in the meanwhile, diluting the melting effect

In more extreme cases the baskets held can be an extremely low number that might even cause the haircut to fail due to `exchangeRateIsValidAfter` modifier on `setBasketsNeeded()`. This would mean trading would be disabled till somebody sends enough balance to the undercollateralized asset.





## PoC

Consider the following scenario:
* A basket is composed of 30 USDc and 1 ETH
* The prices are:
    * 1 USDc = 1 USD
    * ETH = 1500 USD
* Therefore the total basket value is 1515 USD
* Protocol holds 1000 baskets
* Governance changes the USDC quantity to 30 USDC
* Baskets held now is only 500, since we hold only 15K USDC
* Bottom range would be `basketsHeld + (excess_ETH * ETH_lowPrice / basket_highPrice) = 500 + (1500 * 500 * 0.99 / (1530 * 1.01)) = 980`
* Top range would be `basketsHeld + (excess_ETH * ETH_highPrice / basket_lowPrice) = 500 + (1500 * 500 * 1.01 / (1530 * 0.99)) = 1000`
* This is clearly a wrong estimation, which would lead to a haircut of 50% (!) rather than going for a trade.

Note: I mentioned governance change for simplicity, but this can also happen without governance intervention when a collateral gets disabled, it's value declines and a backup asset kicks in (at first the disabled asset would get traded and cover up some of the deficit and then we'd go for a haircut)

## Mitigation
* A more efficient formula would be to use the max baskets held (i.e. the maximum of (each collateral balance divided by the basket_quantity of that collateral)) and then subtract from that the lowest estimation of baskets missing (i.e. lowest value estimation of needed assets to reach that amount divided by highest estimation of basket value).
    * In the case above that would mean `maxBasketsHeld - (USDC_deficit * USDC_lowPrice / basket_highPrice) = 1000 - (500 * 30 * 0.99 / (1530 * 1.01)) = 990.4`. Freeing up 9.6 ETH for sale
* The suggested formula might get us a higher top range estimation when the case is the other way around (the collateral that makes the larger part of the basket value is missing, in our case ETH is missing and USDC not), but it wouldn't result in a haircut and still go for trading (since the top range would be closer to baskets held)


Even with the mitigation above there can be extreme cases where a single asset holds a very small fraction of the total basket value (e.g. 0.1%) and the mitigation wouldn't help much in this case. There might be a need to come up with a broader mitigation for the issue that haircut is done to the number of baskets held rather than bottom range even though the difference between the two can be significant. Or set a threshold for the fraction of the value that each collateral holds in the total value of the basket.
