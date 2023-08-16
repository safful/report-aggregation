## Tags

- bug
- disagree with severity
- 1 (Low Risk)
- sponsor confirmed
- filed

# [getAnchorPrice potentially returns the wrong median](https://github.com/code-423n4/2021-04-vader-findings/issues/213) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The `Router.getAnchorPrice` sorts the `arrayPrices` array and always returns the third element `_sortedAnchorFeed[2]`.
This only returns the median if `_sortedAnchorFeed` is of length 5, but it can be anything from `0` to `anchorLimit`.

## Impact

If not enough anchors are listed initially, it might become out-of-bounds and break all contract functionality due to revert, or return a wrong median.
If `anchorLimit` is set to a different value than 5, it's also wrong.

## Recommended Mitigation Steps

Check the length of `_sortedAnchorFeed` and return `_sortedAnchorFeed[_sortedAnchorFeed.length / 2]` if it's odd, or the average of the two in the middle if it's even.


