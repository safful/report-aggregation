## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [getMostRecentMarket can revert](https://github.com/code-423n4/2021-08-realitycards-findings/issues/13) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function getMostRecentMarket of RCFactory.sol will revert if no markets of the specific mode are created yet.

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCFactory.sol#L171
    function getMostRecentMarket(IRCMarket.Mode _mode)  external view override  returns (address)
    {
        return marketAddresses[_mode][marketAddresses[_mode].length - (1)];
    }


## Tools Used

## Recommended Mitigation Steps
Change the function getMostRecentMarket to something like:

function getMostRecentMarket(IRCMarket.Mode _mode)  external view override  returns (address) {
     if ( marketAddresses[_mode].length ==0) return address(0);
     return marketAddresses[_mode][marketAddresses[_mode].length - (1)];
}


