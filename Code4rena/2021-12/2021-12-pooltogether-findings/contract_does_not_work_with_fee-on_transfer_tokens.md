## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Contract does not work with fee-on transfer tokens](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/30) 

# Handle

pmerkleplant


# Vulnerability details

## Impact

There exist ERC20 tokens that charge a fee for every transfer.

This kind of token does not work correctly with the `TwabRewards` contract as the
rewards calculation for an user is based on `promotion.tokensPerEpoch` (see line [320](https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L320)).

However, the actual amount of tokens the contract holds could be less than
`promotion.tokensPerEpoch * promotion.numberOfEpochs` leading to not claimable
rewards for users claiming later than others.

## Recommended Mitigation Steps

To disable fee-on transfer tokens for the contract, add the following code in
`createPromotion` around line 11:
```
uint256 oldBalance = _token.balanceOf(address(this));
_token.safeTransferFrom(msg.sender, address(this), _tokensPerEpoch * _numberOfEpochs);
uint256 newBalance = _token.balanceOf(address(this));
require(oldBalance + _tokenPerEpoch * _numberOfEpochs == newBalance);
```

