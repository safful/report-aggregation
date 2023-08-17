## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Token: Founder percentages not always respected](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/107) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/debe9b792cc70510eadf9b3728cde5b0f2ec9a1f/src/token/Token.sol#L110


# Vulnerability details

## Impact
Because of the "greedy" minting scheme for founders (tokens to founders are minted until `_isForFounder` returns `false`, i.e. until there is an unset `tokenRecipient[tokenId % 100]`), it can happen that the actual percentages of tokens that the founders receive deviate significantly from the desired percentages:

##  Proof Of Concept
Imagine we are in a situation where one founder has a 51% share and the other a 48% share. Because `schedule` is set to 1 for the first founder, `tokenRecipient[0] ... tokenRecipient[50]` will be set to his address. `tokenRecipient[51], tokenRecipient[53], ...` is set to the address of the second founder. Now let's say a mint happens just before the `vestExpiry` and when `tokenId % 100 == 0`. In such a situation, founder 1 will get 51 tokens (because of the consecutive entries in `tokenRecipients`) and founder 2 will get 1 token (because of the entry in `tokenRecipient[51]`, which is also consecutive. Let's say that the next mint happens after the vest expiration, which means that no founders get additional tokens.

In such a situation, founder 1 got 51 of the "last 100" token IDs, whereas founder 2 only got 1. Therefore, the overall percentage of tokens that those founders got will not be 51% and 40%. When the vest expiration was set to a time far in the future, it will be close to it, but when the vest timespan was only short, it can be very bad. In the extreme case where the expiration is set such that only 1 mint call causes mints for founders, founder 1 will have 51 tokens and founder 2 only 1, meaning the percentages are 51% / 1% instead of 51% / 48%!

## Recommended Mitigation Steps
Consider using another distribution scheme. Instead of the current "greedy" scheme (minting until a slot is free), it would make sense to mint the tokens for the founders every 100 tokens, i.e. everytime when `tokenId % 100 == 0`. Like that, it is ensured that the actual percentages are equal to the desired percentages.