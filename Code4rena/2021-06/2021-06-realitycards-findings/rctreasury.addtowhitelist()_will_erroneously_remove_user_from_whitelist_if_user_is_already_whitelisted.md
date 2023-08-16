## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [RCTreasury.addToWhitelist() will erroneously remove user from whitelist if user is already whitelisted](https://github.com/code-423n4/2021-06-realitycards-findings/issues/49) 

# Handle

jvaqa


# Vulnerability details

RCTreasury.addToWhitelist() will erroneously remove user from whitelist if user is already whitelisted

## Impact

The comments state that calling addToWhitelist() should add a user to the whitelist. [1]

However, since the implementation simply flips the user's whitelist bool, if the user is already on the whitelist, then calling addToWhitelist() will actually remove them from the whitelist. [2]

Since batchAddToWhitelist() will repeatedly call addToWhitelist() with an entire array of users, it is very possible that someone could inadvertently call addToWhitelist twice for a particular user, thereby leaving them off of the whitelist. [3] 

## Proof of Concept

If a governor calls addToWhitelist() with the same user twice, the user will not be added to the whitelist, even though the comments state that they should.

## Recommended Mitigation Steps

Change addToWhitelist to only ever flip a user's bool to true. To clarify the governor's intention, create a corresponding removeFromWhitelist and batchRemoveFromWhitelist which flip a user's bool to false, so that the governor does not accidently remove a user when intending to add them. 

Change this:

isAllowed[_user] = !isAllowed[_user]; // [4]

To this:

isAllowed[_user] = true; // [4]

And add this:

    /// @notice Remove a user to the whitelist
    function removeFromWhitelist(address _user) public override {
        IRCFactory factory = IRCFactory(factoryAddress);
        require(factory.isGovernor(msgSender()), "Not authorised");
        isAllowed[_user] = false;
    }

    /// @notice Remove multiple users from the whitelist
    function batchRemoveFromWhitelist(address[] calldata _users) public override {
        for (uint256 index = 0; index < _users.length; index++) {
            removeFromWhitelist(_users[index]);
        }
    }


[1] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L209

[2] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L213

[3] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L217

[4] https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCTreasury.sol#L213


