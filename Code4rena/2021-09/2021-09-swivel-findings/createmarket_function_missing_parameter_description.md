## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [createMarket function missing parameter description](https://github.com/code-423n4/2021-09-swivel-findings/issues/3) 

# Handle

loop


# Vulnerability details

the parameter `uint8 d` of the `createMarket` function is lacking a parameter description in function spec. 
 
## Impact
No direct impact, but with the parameter naming scheme of only using the first letter of its description the parameter spec is essential.

## Proof of Concept
Code snippet for funciton spec + declaration:
```
/// @notice Allows the owner to create new markets
/// @param u Underlying token address associated with the new market
/// @param m Maturity timestamp of the new market
/// @param c cToken address associated with underlying for the new market
/// @param n Name of the new zcToken market
/// @param s Symbol of the new zcToken market
function createMarket(
    address u,
    uint256 m,
    address c,
    string memory n,
    string memory s,
    uint8 d
) public onlyAdmin(admin) returns (bool)
```

## Recommended Mitigation Steps
Add parameter spec for `uint8 d`

