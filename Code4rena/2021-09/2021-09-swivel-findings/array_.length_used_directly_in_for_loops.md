## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Array .length Used Directly In For Loops](https://github.com/code-423n4/2021-09-swivel-findings/issues/15) 

# Handle

ye0lde


# Vulnerability details

## Impact

There is additional gas usage when an array's length value is used directly in a "for" loop.

## Proof of Concept

The array's length value is used directly in a for loop here:
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L57
https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L211


## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Change the loops above from:
<code>
for (uint256 i=0; i < o.length; i++)
</code>

to
<code>
unit256 length = o.length;
for (uint256 i=0; i < length; i++)
</code>

When I tested these changes there was a small gas saving.


