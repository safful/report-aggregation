## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`BasketLicenseProposed` better emit proposal id](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/134) 

# Handle

gzeon


# Vulnerability details

## Impact
Since tokenName is user supplied and can be duplicated, it is better to emit proposal id instead.

https://github.com/code-423n4/2021-12-defiprotocol/blob/205d3766044171e325df6a8bf2e79b37856eece1/contracts/contracts/Factory.sol#L91
```
        emit BasketLicenseProposed(msg.sender, tokenName);
```

