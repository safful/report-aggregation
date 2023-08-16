## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing timelock for critical contract setters of privileged roles (Price Oracles)](https://github.com/code-423n4/2021-12-sublime-findings/issues/103) 

# Handle

defsec


# Vulnerability details

## Impact

Setter functions for critical contract parameters accessible only by privileged roles e.g. admin should consider adding timelocks (along with emitted events) so that users and other privileged roles can detect upcoming changes and have the time to react to them.

Changes to whitelists, oracle addresses and migrator address may have a financial or trust impact on users who should be given an opportunity to react to them by exiting/engaging without being surprised when such changes are made effective immediately.

See similar Medium-severity finding in ConsenSys's Audit of 1inch Liquidity Protocol (https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unpredictable-behavior-for-users-due-to-admin-front-running-or-general-bad-timing)



## Proof of Concept

1. Navigate to the following contract.

https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/PriceOracle.sol#L189

https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/PriceOracle.sol#L203

2. The functions are responsible for the price oracles. Therefore, If the price oracle is set to wrong. All price feeds will be affected by this. 

## Tools Used

None

## Recommended Mitigation Steps

Consider adding timelocks to such contracts with critical setter functions.



