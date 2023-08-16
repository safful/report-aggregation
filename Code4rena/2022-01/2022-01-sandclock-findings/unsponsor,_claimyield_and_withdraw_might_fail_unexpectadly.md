## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- sponsor vault

# [unsponsor, claimYield and withdraw might fail unexpectadly](https://github.com/code-423n4/2022-01-sandclock-findings/issues/76) 

# Handle

danb


# Vulnerability details

`totalUnderlying()` includes the invested assets, they are not in the contract balance.

when a user calls withdraw, claimYield or unsponsor, the system might not have enough assets in the balance and the transfer would fail.

especially, force unsponsor will always fail, because it tries to transfer the entire `totalUnderlying()`, which the system doesn't have:

https://github.com/code-423n4/2022-01-sandclock/blob/main/sandclock/contracts/Vault.sol#L391


## Recommended Mitigation Steps
when the system doesn't have enough balance to make the transfer, withdraw from the strategy.

