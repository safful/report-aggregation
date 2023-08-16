## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Tokens not recoverable](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/108) 

# Handle

Reigada


# Vulnerability details

## Impact
In both airdrop contracts: AirdropDistribution and InvestorDistribution, once all the participants have claimed their tokens, some will remain in the contract due to some imprecision in the calculations. There is no function that allows to substract them which means that those tokens will remain stuck in the contracts forever.

I have made the test with just 5 participants: user2, user3, user4, user5 & user6.

uint256[5] airdropBalances =
    [
    4032000,
    4032000,
    4032000,
    4032000,
    4032000
    ];

>>> 4032000 * 5
20160000

Initially 20160000 tokens were transferred to the contract 
mockToken.transfer(airdropdist.address, 20160000000000000000000000)

After 260 weeks, these were the results:

----------------> mockToken.balanceOf(airdropdist.address) -> 2842805668532461833600 <-----------------

mockToken.balanceOf(user2) -> 1209429431659888052289865
vesting.benTotal(user2.address) -> 2822002007206405455343415
mockToken.balanceOf(user3) -> 1209429431659888052289984
vesting.benTotal(user3.address) -> 2822002007206405455343296
mockToken.balanceOf(user4) -> 1209429431659888052289984
vesting.benTotal(user4.address) -> 2822002007206405455343296
mockToken.balanceOf(user5) -> 1209429431659888052289984
vesting.benTotal(user5.address) -> 2822002007206405455343296
mockToken.balanceOf(user6) -> 1209429431659888052289984
vesting.benTotal(user6.address) -> 2822002007206405455343296

As we can see above 2842 tokens remain in the contract and there is no way to retrieve them.

## Tools Used
Manual testing / brownie

## Recommended Mitigation Steps
Add an onlyOwner function that allows to retrieve all the remaining tokens once all the participants of the airdrop have claimed the whole amount of their rewards.

