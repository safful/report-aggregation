## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [transferNotionalFrom doesn't check from != to](https://github.com/code-423n4/2021-09-swivel-findings/issues/65) 

# Handle

gpersoon


# Vulnerability details

# Impact
The function transferNotionalFrom of VaultTracker.sol uses temporary variables to store the balances.
If the "from" and "to" address are the same then the balance of "from" is overwritten by the balance of "to".
This means the balance of "from" and "to" are increased and no balances are decreased, effectively printing money.

Note: transferNotionalFrom can be called via transferVaultNotional by everyone.

## Proof of Concept
https://github.com/Swivel-Finance/gost/blob/v2/test/vaulttracker/VaultTracker.sol#L144-L196

 function transferNotionalFrom(address f, address t, uint256 a) external onlyAdmin(admin) returns (bool) {
    Vault memory from = vaults[f];
    Vault memory to = vaults[t];
    ...
    vaults[f] = from;
    ...
    vaults[t] = to;    // if f==t then this will overwrite vaults[f] 


https://github.com/Swivel-Finance/gost/blob/v2/test/marketplace/MarketPlace.sol#L234-L238
function transferVaultNotional(address u, uint256 m, address t, uint256 a) public returns (bool) {
    require(VaultTracker(markets[u][m].vaultAddr).transferNotionalFrom(msg.sender, t, a), 'vault transfer failed');
   
## Tools Used

## Recommended Mitigation Steps
Add something like the following:
   require (f != t,"Same");

