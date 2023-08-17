## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-07-yield-findings/issues/98) 

Gas saving:

Handling validation check
Condition check could be  >=  in below line of code. This could save gas by skipping the calculation done in the else part.
 https://github.com/code-423n4/2022-07-yield/blob/6ab092b8c10e4dabb470918ae15c6451c861655f/contracts/Witch.sol#L586
When elapsed==duration, below calculation will always return 1e18
https://github.com/code-423n4/2022-07-yield/blob/6ab092b8c10e4dabb470918ae15c6451c861655f/contracts/Witch.sol#L588-L592

memory can be used instead of storage in following lines of codes
https://github.com/code-423n4/2022-07-yield/blob/6ab092b8c10e4dabb470918ae15c6451c861655f/contracts/Witch.sol#L254
https://github.com/code-423n4/2022-07-yield/blob/6ab092b8c10e4dabb470918ae15c6451c861655f/contracts/Witch.sol#L231