## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Owners receive more percentage of total nft if some nfts were burned(because were not sold)](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L71-L126
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L154
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L207-L213


# Vulnerability details

## Impact
According to nouns builder, founder can have percentage of created nft. This is set in `Token::_addFounders` function. 
When new nft is minted by `mint` function then total supply of tokens is incremented and assigned to tokenId using `tokenId = settings.totalSupply++`. Then this token is checked if it should be mint to founder(then again increment total supply of tokens) or should be mint to auction using `while (_isForFounder(tokenId))`.

If token wasn't sold during the auction then auction burns it using `burn` function. And this function doesn't decrement `settings.totalSupply` value. But total supply **has changed** now, it has decreased by one.

So suppose that we have 1 founder of dao that should receive 2% of nft, that means that if 100 nft are available(for example), then 2 of them belongs to that founder. If we have minted 100 nft and 10 of them were not sold(they were then burned), then there are 90 nft available now. And in current implementation founder has ownership of 2 of them, however **2 is not 2% of 90**. So in case when nft are not sold on auction the percentage of founder's tokens is increasing and the increasing speed depends on how many tokens were not sold. Also founder gets more power in the community(as he has more percentage now). 

## Proof of Concept
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L71-L126
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L154
https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/Token.sol#L207-L213

## Tools Used

## Recommended Mitigation Steps
When `burn` function is called then do `settings.totalSupply--`.