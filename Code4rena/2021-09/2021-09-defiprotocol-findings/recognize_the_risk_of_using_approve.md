## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Recognize the risk of using approve](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/190) 

# Handle

0xRajeev


# Vulnerability details

## Impact

While safeApprove is used in the Factory contract, the use of ERC20 approve in approveUnderlying() (instead of safeApprove) is presumably to handle the reapprovals during changing of index but is susceptible to the historical ERC20 approve() race condition.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L226

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L106

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Be aware that this is susceptible to race-condition but this it unlikely a concern because the spender is always the auction contract which is cloned and therefore trusted.

