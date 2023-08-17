## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-11

# [`_handleOpenFees` returns an incorrect value for `_feePaid`. This directly impacts margin calculations](https://github.com/code-423n4/2022-12-tigris-findings/issues/367) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L178
https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L734


# Vulnerability details

## Impact

Formula for `fee paid` in [Line 734](https://github.com/code-423n4/2022-12-tigris/blob/main/contracts/Trading.sol#L734) is incorrect leading to incorrect margin calculations. Since this directly impacts the trader margin and associated fee calculations, I've marked as HIGH risk

On initiating a market order, `Margin` is adjusted for the `fees` that is charged by protocol. This adjustment is in [Line 178 of Trading](https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L178). Fees computed by `_handleOpenFees ` is deducted from Initial margin posted by user.

formula misses to account the `2*referralFee` component while calculaing `_feePaid`

## Proof of Concept
Note that `_feePaid` as per formula in Line 734 is the sum of `_daoFeesPaid', and sum of `burnerFee` & `botFee`. `_daoFeesPaid` is calculated from `_fees.daoFees` which itself is calculated by subtracting `2*referralFee` and `botFee`. 

So when we add back `burnerFee` and `botFee` to `_feePaid`, we are missing to add back the `2*referralFee`  which was earlier excluded when calculating `_daoFeesPaid`. While `botFee` is added back correctly, same adjustment is not being done viz-a-viz referral fee.

 This results in under calculating the `_feePaid` and impacts the rewards paid to the protocol NFT holders.


## Tools Used

## Recommended Mitigation Steps

Suggest replacing the formula in line 734 with below (adding back _fees.referralFees*2)

```
            _feePaid =
                _positionSize
                * (_fees.burnFees + _fees.botFees + _fees.referralFees*2 ) 
                / DIVISION_CONSTANT // divide by 100%
                + _daoFeesPaid;
```
