## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Deny of service in `AccountantDelegate.sweepInterest`](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/28) 

# Lines of code

https://github.dev/Plex-Engineer/lending-market-v2/blob/2646a7676b721db8a7754bf5503dcd712eab2f8a/contracts/Accountant/AccountantDelegate.sol#L101


# Vulnerability details

## Impact
The `sweepInterest` method is susceptible to denial of service.

## Proof of Concept
The logic of the `sweepInterest` method relative to the `treasury` is as follows:
```javascript
		bool success = cnote.transfer(treasury, amtToSweep);
		if (!success) { revert  SweepError(treasury , amtToSweep); }
		TreasuryInterface Treas = TreasuryInterface(treasury);
		Treas.redeem(address(cnote),amtToSweep);
		require(cnote.balanceOf(treasury) == 0, "AccountantDelegate::sweepInterestError");
```

As you can see, `amtToSweep` is passed to it and `redeem` that amount. Later it is checked that the balance of `cnote` in the `treasury` address must be 0. However, all calculations related to `amtToSweep` come out of the balance of [address(this)](https://github.dev/Plex-Engineer/lending-market-v2/blob/2646a7676b721db8a7754bf5503dcd712eab2f8a/contracts/Accountant/AccountantDelegate.sol#L83-L84) so if a third party sends a single token `cnote` to the address of `treasury` the method will be denied.

## Recommended Mitigation Steps
- Check that the balance is the same after and before the `bool success = cnote.transfer(treasury, amtToSweep);`

