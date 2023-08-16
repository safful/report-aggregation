## Tags

- bug
- 0 (Non-critical)
- SwappableYieldSource
- sponsor confirmed

# [Possible enhancements to supply/redeem full balance](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/79) 

# Handle

pauliax


# Vulnerability details

## Impact
Consider adding functions in SwappableYieldSource to supply/redeem the whole balance of the user, so users will not need to pass an exact amount in case they want to fully join/exit the pool. 
Also, you can consider joining the BoostedVault for some extra rewards, however, I think then funds will need to be locked for some time for the rewards to start accruing.


