## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [function build could explicitly check that seriesId is not 0](https://github.com/code-423n4/2021-05-yield-findings/issues/59) 

# Handle

pauliax


# Vulnerability details

## Impact
It would be helpful if function build explicitly check that seriesId != bytes12(0). In practice, it is not possible to have a series with an id of 0, so this check will not pass:
    require (ilks[seriesId][ilkId] == true, "Ilk not added to series");
however, the error message is not very informative, thus I am suggesting adding an explicit check.

## Recommended Mitigation Steps
require (seriesId != bytes12(0), "Series id is zero");

