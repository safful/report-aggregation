## Tags

- mitigation-confirmed
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- MR-M-04

# [Mitigation of M-04: Issue not fully mitigated](https://github.com/code-423n4/2023-02-reserve-mitigation-contest-findings/issues/54) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/610cfca553beea41b9508abbfbf4ee4ce16cbc12/contracts/p1/RToken.sol#L215


# Vulnerability details

### Original issue: [M-04: Redemptions during undercollateralization can be hot-swapped to steal all funds)](https://github.com/code-423n4/2023-01-reserve-findings/issues/399)

## Impact
User might be agreeing to a partial redemption expecting to lose only a small fraction, but end up losing a significantly higher fraction.

### Details

* Issue was that user might get only partial redemption when they didn't intend to - they sent a tx to the pool, in the meanwhile an asset got disabled and replaced. Redeemer doesn't get any share of the disabled collateral, and the backup collateral balance is zero.
* Mitigation adds a parameter named `revertOnPartialRedemption`, if the parameter is false the redeeming would revert if any collateral holds only part of the asset.
* This is suppose to solve this issue since in the case above the user would set it to false and the redeeming tx would revert
* The issue is that there might be a case where the protocol holds only a bit less than the quantity required (e.g. 99%), and in that case the user would be setting `revertOnPartialRedemption` to true, expecting to get 99% of the value of the basket. Then if an asset is disabled and replaced the user would suffer a loss much greater than they've agreed to.

## PoC
### Likelihood
Mostly the protocol wouldn't be undercollateralized for a long time, since there would either by trading going on to cover it or eventually there would be a haircut. But there can still be periods of time where this happens:
* Governance increased the basket quantity of one asset a bit (expecting the yield to cover for it), trading won't start till `tradingDelay` passes. Meaning a few hours where only partial redemption would be possible.
* Another asset got disabled first, and replaced by a backup asset. The protocol either had enough balance of the backup asset or covered up for it via trading. Yet again, this won't last long since eventually all trading would complete and the protocol would go to a haircut, but there can be multiple trading of multiple assets which would make it last up to a few hours.


## Mitigation
The ideal solution would be to allow the user to specify the min amount for each asset or the min ratio of the between the total redemption value and the basket value, but that would be too expensive and complicated.
I think the middle way here would be to replace `revertOnPartialRedemption` parameter with a single numeric parameter that specifies the min ratio that the user expects to get (i.e. if that parameter is set to 90%, that means that if any asset holds less than 90% than the quantity it should the redemption would revert).
This shouldn't cost much more gas, and would cover most of the cases.