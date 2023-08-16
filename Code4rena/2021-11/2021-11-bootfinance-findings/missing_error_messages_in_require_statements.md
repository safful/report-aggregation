## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Missing error messages in require statements](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/247) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/vesting/contracts/AirdropDistribution.sol#L525-L525
```solidity=525
require(airdrop[msg.sender].amount != 0);
```

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/vesting/contracts/AirdropDistribution.sol#L561-L561
```solidity=561
require(airdrop[msg.sender].amount >= claimable);
```

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/vesting/contracts/InvestorDistribution.sol#L100-L100
```solidity=100
require(investors[_investor].amount != 0);
```

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/vesting/contracts/InvestorDistribution.sol#L126-L126
```solidity=126
require(investors[msg.sender].amount - claimable != 0);
```


