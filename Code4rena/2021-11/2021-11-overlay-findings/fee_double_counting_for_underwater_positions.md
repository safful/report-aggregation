## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Fee double counting for underwater positions](https://github.com/code-423n4/2021-11-overlay-findings/issues/134) 

# Handle

hyh


# Vulnerability details

## Impact

Actual available fees are less than recorded. That's because a part of them corresponds to underwater positions, and will not have the correct amount stored with the contract: when calculation happens the fee is recorded first, then there is a check for position health, and the funds are channeled to cover the debt firsthand. This way in a case of unfunded position the fee is recorded, but cannot be allocated, so the fees accounted can be greater than value of fees stored.

This can lead to fee withdrawal malfunction, i.e. disburse() will burn more and attempt to transfer more than needed. This leads either to inability to withdraw fees when disburse be failing due to lack of funds, or funds leakage to fees and then inability to perform other withdrawals because of lack of funds.

## Proof of Concept

The fees are accounted for before position health check and aren't corrected thereafter when there is a shortage of funds.

https://github.com/code-423n4/2021-11-overlay/blob/main/contracts/collateral/OverlayV1OVLCollateral.sol#L311

## Recommended Mitigation Steps

Adjust fees after position health check: accrue fees only on a remaining part of position that is available after taking debt into account.

Now:
```
uint _feeAmount = _userNotional.mulUp(mothership.fee());

uint _userValueAdjusted = _userNotional - _feeAmount;
if (_userValueAdjusted > _userDebt) _userValueAdjusted -= _userDebt;
else _userValueAdjusted = 0;
```

To be:
```
uint _feeAmount = _userNotional.mulUp(mothership.fee());

uint _userValueAdjusted = _userNotional - _feeAmount;
if (_userValueAdjusted > _userDebt) {
	_userValueAdjusted -= _userDebt;
} else {
	_userValueAdjusted = 0;
	_feeAmount = _userNotional > _userDebt ? _userNotional - _userDebt : 0;
}
```


