## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- old-submission-method
- selected-for-report

# [GolomTrader's _settleBalances double counts protocol fee, reducing taker's payout for a NFT sold](https://github.com/code-423n4/2022-07-golom-findings/issues/240) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L375-L400


# Vulnerability details

Currently `(o.totalAmt * 50) / 10000)` protocol fee share is multiplied by `amount` twice when being accounted for as a deduction from the total in amount due to the `msg.sender` taker calculations in _settleBalances(), which is called by fillBid() and fillCriteriaBid() to handle the payouts.

Setting the severity to be high as reduced payouts is a fund loss impact for taker, which receives less than it's due whenever `amount > 1`.

Notice that the amount lost to the taker is left on the contract balance and currently is subject to other vulnerabilities, i.e. can be easily stolen by an attacker that knowns these specifics and tracks contract state. When these issues be fixed this amount to be permanently frozen on the GolomTrader's balance as it's unaccounted for in all subsequent calculations (i.e. all the transfers are done with regard to the accounts recorded, this extra sum is unaccounted, there is no general native funds rescue function, so when all other mechanics be fixed the impact will be permanent freeze of the part of taker's funds).

## Proof of Concept

_settleBalances() uses `(o.totalAmt - protocolfee - ...) * amount`, which is `o.totalAmt * amount - ((o.totalAmt * 50) / 10000) * amount * amount - ...`, counting protocol fee extra `amount - 1` times:

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L389-L399

```solidity
            payEther(
                (o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt - o.refererrAmt) *
                    amount -
                    p.paymentAmt,
                msg.sender
            );
        } else {
            payEther(
                (o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt) * amount - p.paymentAmt,
                msg.sender
            );
```

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L375-L400

```solidity
    function _settleBalances(
        Order calldata o,
        uint256 amount,
        address referrer,
        Payment calldata p
    ) internal {
        uint256 protocolfee = ((o.totalAmt * 50) / 10000) * amount;
        WETH.transferFrom(o.signer, address(this), o.totalAmt * amount);
        WETH.withdraw(o.totalAmt * amount);
        payEther(protocolfee, address(distributor));
        payEther(o.exchange.paymentAmt * amount, o.exchange.paymentAddress);
        payEther(o.prePayment.paymentAmt * amount, o.prePayment.paymentAddress);
        if (o.refererrAmt > 0 && referrer != address(0)) {
            payEther(o.refererrAmt * amount, referrer);
            payEther(
                (o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt - o.refererrAmt) *
                    amount -
                    p.paymentAmt,
                msg.sender
            );
        } else {
            payEther(
                (o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt) * amount - p.paymentAmt,
                msg.sender
            );
        }
```

Say, if `amount = 6`, while `((o.totalAmt * 50) / 10000) = 1 ETH`, `6 ETH` is total `protocolfee` and needs to be removed from `o.totalAmt * 6` to calculate taker's part, while `1 ETH * 6 * 6 = 36 ETH` is actually removed in the calculation, i.e. `36 - 6 = 30 ETH` of taker's funds will be frozen on the contract balance.

## Recommended Mitigation Steps

Consider accounting for `amount` once, for example:

https://github.com/code-423n4/2022-07-golom/blob/e5efa8f9d6dda92a90b8b2c4902320acf0c26816/contracts/core/GolomTrader.sol#L375-L403

```solidity
    function _settleBalances(
        Order calldata o,
        uint256 amount,
        address referrer,
        Payment calldata p
    ) internal {
-       uint256 protocolfee = ((o.totalAmt * 50) / 10000) * amount;
+       uint256 protocolfee = ((o.totalAmt * 50) / 10000);
        WETH.transferFrom(o.signer, address(this), o.totalAmt * amount);
        WETH.withdraw(o.totalAmt * amount);
-       payEther(protocolfee, address(distributor));
+       payEther(protocolfee * amount, address(distributor));
        payEther(o.exchange.paymentAmt * amount, o.exchange.paymentAddress);
        payEther(o.prePayment.paymentAmt * amount, o.prePayment.paymentAddress);
        if (o.refererrAmt > 0 && referrer != address(0)) {
            payEther(o.refererrAmt * amount, referrer);
            payEther(
                (o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt - o.refererrAmt) *
                    amount -
                    p.paymentAmt,
                msg.sender
            );
        } else {
            payEther(
                (o.totalAmt - protocolfee - o.exchange.paymentAmt - o.prePayment.paymentAmt) * amount - p.paymentAmt,
                msg.sender
            );
        }
        payEther(p.paymentAmt, p.paymentAddress);
-       distributor.addFee([msg.sender, o.exchange.paymentAddress], protocolfee);
+       distributor.addFee([msg.sender, o.exchange.paymentAddress], protocolfee * amount);
    }
```

