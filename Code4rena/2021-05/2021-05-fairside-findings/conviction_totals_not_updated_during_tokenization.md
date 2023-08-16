## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Conviction totals not updated during tokenization](https://github.com/code-423n4/2021-05-fairside-findings/issues/28) 

# Handle

0xRajeev


# Vulnerability details

## Impact

_updateConvictionScore() function returns convictionDelta and governanceDelta which need to be used immediately in a call to _updateConvictionTotals(convictionDelta, governanceDelta) for updating the conviction totals of conviction and governance-enabled conviction for the entire FairSide network.

This updation of totals after a call to _updateConvictionScore() is done on Line70 in _beforeTokenTransfer() and Line367 in updateConvictionScore() of ERC20ConvictionScore.sol.

However, the return values of _updateConvictionScore() are ignored on Line284 in tokenizeConviction() and not used to update the totals using _updateConvictionTotals(convictionDelta, governanceDelta).

The impact is that when a user tokenizes their conviction score, their conviction deltas are updated and recorded (only if the funds locked are zero which is incorrect and reported separately in a different finding) but the totals are not updated. This leads to incorrect accounting of TOTAL_CONVICTION_SCORE and TOTAL_GOVERNANCE_SCORE which are used in the calculation of tributes and therefore will lead to incorrect tribute calculations.

## Proof of Concept

Alice calls tokenizeConviction() to convert her conviction score into an NFT. Her conviction deltas as returned by _updateConvictionScore() are ignored and TOTAL_CONVICTION_SCORE and TOTAL_GOVERNANCE_SCORE values are not updated. As a result, the tributes rewarded are proportionally more than what should have been the case because the conviction score totals are used as the denominator in availableTribute() and availableGovernanceTribute().

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L284

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L108-L110

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L52-L70

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L365-L367

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/ERC20ConvictionScore.sol#L73-L106

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/TributeAccrual.sol#L83-L100

https://github.com/code-423n4/2021-05-FairSide/blob/3e9f6d40f70feb67743bdc70d7db9f5e3a1c3c96/contracts/dependencies/TributeAccrual.sol#L102-L123


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use the return values of _updateConvictionScore() function (i.e. convictionDelta and governanceDelta) on Line284 of ERC20ConvictionScore.sol and use them in a call to _updateConvictionTotals(convictionDelta, governanceDelta).

