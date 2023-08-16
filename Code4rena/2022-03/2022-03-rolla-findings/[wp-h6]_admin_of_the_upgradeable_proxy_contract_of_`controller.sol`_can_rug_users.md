## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [[WP-H6] Admin of the upgradeable proxy contract of `Controller.sol` can rug users](https://github.com/code-423n4/2022-03-rolla-findings/issues/48) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/a06418c9cc847395f3699bdf684a9ac066651ed7/quant-protocol/contracts/Controller.sol#L22-L34


# Vulnerability details

Use of Upgradeable Proxy Contract Structure allows the logic of the contract to be arbitrarily changed.

This allows the proxy admin to perform malicious actions e.g., taking funds from users' wallets up to the allowance limit.

This action can be performed by the malicious/compromised proxy admin without any restriction. 

Considering that the purpose of this particular contract is for accounting of the Collateral and LongShortTokens, we believe the users' allowances should not be hold by this upgradeable contract.

### PoC

Given:

- collateral: `USDC`

#### Rug Users' Allowances

1. Alice `approve()` and `_mintOptionsPosition()` with `1e8 USDC`;
2. Bob  `approve()` and `_mintOptionsPosition()` with `5e8 USDC`;
3. A malicious/compromised proxy admin can call `upgradeToAndCall()` on the proxy contract and set a malicious contract as `newImplementation` and stolen all the USDC in Alice and Bob's wallets;

#### Rug Contract's Holdings (funds that belongs to users)

A malicious/compromised proxy admin can just call `upgradeToAndCall()` on the proxy contract and send all the USDC held by the contract to an arbitrary address.

### Severity

A smart contract being structured as an upgradeable contract alone is not usually considered as a high severity risk. But given the severe impact (all the funds in the contract and funds in users' wallets can be stolen), we mark it as a `High` severity issue.

### Recommendation

Consider using the non-upgradeable `CollateralToken` contract to hold user's allowances instead.

See also the Recommendation of [WP-H7].

