## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Missing `_from` param comment on `LivepeerToken.sol:burn()`](https://github.com/code-423n4/2022-01-livepeer-findings/issues/140) 

# Handle

Dravee


# Vulnerability details

## Impact
The `_from` parameter comment is missing on `LivepeerToken.sol:burn()`.
The impact is minimal, but as it's commented elsewhere (https://github.com/livepeer/arbitrum-lpt-bridge/blob/af952a58eff5ff84559e25f62e29f2a3d9e176f9/contracts/L2/gateway/L2LPTGateway.sol#L96), I figured I'd mention it.

## Proof of Concept
https://github.com/livepeer/arbitrum-lpt-bridge/blob/e89be1431024d976b8c97bbe64ec4bdfeb28ec64/contracts/L2/token/LivepeerToken.sol#L32-L36
```
File: LivepeerToken.sol
32:     /**
33:      * @dev Burns a specific amount of the sender's tokens
34:      * @param _amount The amount of tokens to be burned
35:      */
36:     function burn(address _from, uint256 _amount)
37:         external
38:         override
39:         onlyRole(BURNER_ROLE)
40:     {
41:         _burn(_from, _amount);
42:         emit Burn(_from, _amount);
43:     }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Add the missing comment

