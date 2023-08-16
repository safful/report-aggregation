## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Deactivate function can be bypassed](https://github.com/code-423n4/2022-02-concur-findings/issues/28) 

# Handle

csanuragjain


# Vulnerability details

## Impact
onlyClient can deactivate a token even after deadline is passed and transfer all token balance to itself

## Proof of Concept
1. Navigate to contract at https://github.com/code-423n4/2022-02-concur/blob/main/contracts/Shelter.sol

2. Observe that token can only be deactivated if activated[_token] + GRACE_PERIOD > block.timestamp. We will bypass this

3. onlyClient activates a token X using the activate function

4. Assume Grace period is crossed such that activated[_token] + GRACE_PERIOD < block.timestamp

5. Now if onlyClient calls deactivate function, it fails with "too late"

6. But onlyClient can bypass this by calling activate function again on token X which will reset the timestamp to latest in activated[_token] and hence onlyClient can now call deactivate function to disable the token and retrieve all funds present in the contract to his own address

## Recommended Mitigation Steps
Add below condition to activate function

```
function activate(IERC20 _token) external override onlyClient {
require(activated[_token]==0, "Already activated");
        activated[_token] = block.timestamp;
        savedTokens[_token] = _token.balanceOf(address(this));
        emit ShelterActivated(_token);
    }
```

