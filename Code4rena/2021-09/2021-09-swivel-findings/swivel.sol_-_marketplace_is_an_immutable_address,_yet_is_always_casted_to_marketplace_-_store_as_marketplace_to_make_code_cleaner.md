## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Swivel.sol - marketplace is an immutable address, yet is always casted to MarketPlace - store as MarketPlace to make code cleaner](https://github.com/code-423n4/2021-09-swivel-findings/issues/125) 

# Handle

GalloDaSballo


# Vulnerability details

## Impact
The variable `marketplace` in Swivel.sol https://github.com/Swivel-Finance/gost/blob/5fb7ad62f1f3a962c7bf5348560fe88de0618bae/test/swivel/Swivel.sol#L21

is stored as an immutable address

However, every single instance of it's usage casts it to `MarketPlace`

Would recommend storing `marketplace` as `MarketPlace` to make the code cleaner


## Recommended Mitigation Steps
Replace
`  address public immutable marketPlace;
`
With
`
  Marketplace public immutable marketPlace;
`

