## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [unpaused(p) modifier missing in authRedeem function](https://github.com/code-423n4/2022-07-swivel-findings/issues/64) 

# Lines of code

https://github.com/code-423n4/2022-07-swivel/blob/main/Marketplace/MarketPlace.sol#L148


# Vulnerability details

## Impact
Due to missing modifier, User will be able to redeem zcTokens and withdraw underlying even in paused Market. This happens due to missing unpaused(p) modifier

## Proof of Concept
1. Lets see function definition for authRedeem function

```
function authRedeem(uint8 p, address u, uint256 m, address f, address t, uint256 a) public authorized(markets[p][u][m].zcToken) returns (uint256 underlyingAmount)
```

2. Observe that unpaused(p) modifier is missing

3. This means if Marketplace is placed under paused state by Admin, then also User can call authRedeem at Marketplace via withdraw/redeem at ZcToken contract. 

4. This will allow Users to withdraw in paused state which is incorrect

## Recommended Mitigation Steps
Add unpaused(p) modifier in authRedeem function

```
function authRedeem(uint8 p, address u, uint256 m, address f, address t, uint256 a) public authorized(markets[p][u][m].zcToken) unpaused(p) returns (uint256 underlyingAmount) {
...
}
```

