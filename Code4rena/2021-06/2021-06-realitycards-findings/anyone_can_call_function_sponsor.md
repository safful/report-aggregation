## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [anyone can call function sponsor](https://github.com/code-423n4/2021-06-realitycards-findings/issues/40) 

# Handle

pauliax


# Vulnerability details

## Impact
This function sponsor should only be called by the factory, however, it does not have any auth checks, so that means anyone can call it with an arbitrary _sponsorAddress address and transfer tokens from them if the allowance is > 0:
    /// @notice ability to add liqudity to the pot without being able to win.
    /// @dev called by Factory during market creation
    /// @param _sponsorAddress the msgSender of createMarket in the Factory
    function sponsor(address _sponsorAddress, uint256 _amount)
        external
        override
    {
        _sponsor(_sponsorAddress, _amount);
    }

## Recommended Mitigation Steps
Check that the sender is a factory contract.

