## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [maxSellAllAmount and maxBuyAllAmount functions can be unintentionally paused (always revert).](https://github.com/code-423n4/2022-05-rubicon-findings/issues/282) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L290
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconRouter.sol#L307


# Vulnerability details

## Impact
The two functions maxSellAllAmount and maxBuyAllAmount will always revert in case at least (100-fee)\% of user's balance can be matched with orders.  

## Proof of Concept
Let say Bob placed an order selling 100 USDC with a low USDT price of 1:0.95.

Alice currently has 50 USDT and they want to maxSellAllAmount into USDC. 

The function will pass 50 as amount into RubiconMarket's buyAll function where it fully matches with Bob's order. Here, the buy() function will first transfer alice's 50 USDT in and later 50 * feeBPS / BPS as fee. In this case, alice can not afford to pay.


Therefore, the two functions maxSellAllAmount and maxBuyAllAmount are useless in case user's request can be fully matched.

## Recommended Mitigation Steps

Add the fee calculating before passing the amount to the RubiconMarket's buyAll, sellAll function. 

```solidity
    /// @dev this function takes a user's entire balance for the trade in case they want to do a max trade so there's no leftover dust
    function maxBuyAllAmount(
        ERC20 buy_gem,
        ERC20 pay_gem,
        uint256 max_fill_amount
    ) external returns (uint256 fill) {
        //swaps msg.sender's entire balance in the trade   
        
        uint256 maxAmount = _calcAmountAfterFee(ERC20(buy_gem).balanceOf(msg.sender));
        
        fill = RubiconMarket(RubiconMarketAddress).buyAllAmount(
            buy_gem,
            maxAmount,
            pay_gem,
            max_fill_amount
        );
        ERC20(buy_gem).transfer(msg.sender, fill);
    }

    /// @dev this function takes a user's entire balance for the trade in case they want to do a max trade so there's no leftover dust
    function maxSellAllAmount(
        ERC20 pay_gem,
        ERC20 buy_gem,
        uint256 min_fill_amount
    ) external returns (uint256 fill) {
        //swaps msg.sender entire balance in the trade

        uint256 maxAmount = _calcAmountAfterFee(ERC20(buy_gem).balanceOf(msg.sender));
        fill = RubiconMarket(RubiconMarketAddress).sellAllAmount(
            pay_gem,
            maxAmount,
            buy_gem,
            min_fill_amount
        );
        ERC20(buy_gem).transfer(msg.sender, fill);
    }


    function _calcAmountAfterFee(uint256 amount) internal view returns (uint256) {
        uint256 feeBPS = RubiconMarket(RubiconMarketAddress).getFeeBPS();
        return amount.sub(amount.mul(feeBPS).div(10000));
    }
```



