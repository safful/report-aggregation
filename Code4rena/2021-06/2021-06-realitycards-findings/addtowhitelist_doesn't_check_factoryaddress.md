## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [addToWhitelist doesn't check factoryAddress](https://github.com/code-423n4/2021-06-realitycards-findings/issues/24) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function addToWhitelist of RCTreasury.sol does a call to the factory contract, however the factoryAddress might not be initialized, because it is set via a different function
(setFactoryAddress).
The function addToWhitelist will revert when it calls a 0 address, but it might be more difficult to troubleshoot.

## Proof of Concept
// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L233
   function setFactoryAddress(address _newFactory) external override {
        ...
        factoryAddress = _newFactory;
    }

// https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L210
    function addToWhitelist(address _user) public override {
        IRCFactory factory = IRCFactory(factoryAddress);
        require(factory.isGovernor(msgSender()), "Not authorised");
        isAllowed[_user] = !isAllowed[_user];
    }

## Tools Used

## Recommended Mitigation Steps
Verify that factoryAddress is set in the function addToWhitelist, for example using the following code.
 require(factory != address(0), "Must have an address");

