## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Prevent Minting During Emergency Exit](https://github.com/code-423n4/2021-11-yaxis-findings/issues/12) 

# Handle

TimmyToes


# Vulnerability details

## Impact
Potential increased financial loss during security incident.

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/Alchemist.sol#L611
Consider a critical incident where a vault is being drained or in danger of being drained due to a vulnerability within the vault or its strategies.
At this stage, you want to trigger emergency exit and users want to withdraw their funds and repay/liquidate to enable the withdrawal of funds. However, minting against debt does not seem like a desirable behaviour at this time. It only seems to enable unaware users to get themselves into trouble by locking up their funds, or allow an attacker to do more damage.

## Recommended Mitigation Steps
Convert emergency exit check to a modifier, award wardens who made that suggestion, and then apply that modifier here.

Alternatively, it is possible that the team might want to allow minting against credit: users minting against credit would effectively be cashing out their rewards. This might be seen as desirable during emergency exit, or it might be seen as a potential extra source of risk. If this is desired, then the emergency exit check could be placed at line 624 with a modified message, instructing users to only use credit.

