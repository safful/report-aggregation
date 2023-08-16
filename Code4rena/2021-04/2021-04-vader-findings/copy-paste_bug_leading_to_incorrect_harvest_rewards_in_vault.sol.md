## Tags

- bug
- disagree with severity
- 2 (Med Risk)
- sponsor confirmed

# [Copy-paste bug leading to incorrect harvest rewards in Vault.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/51) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The conditional in calcReward() function uses the same code in both if/else parts with repeated use of reserveUSDV, reserveVADER and getUSDVAmount leading to incorrect computed value of _adjustedReserve in the else part.

This will affect harvest rewards for all users of the protocol and lead to incorrect accounting. Protocol will break and lead to fund loss.

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vault.sol#L141

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vault.sol#L144

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vault.sol#L125

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vault.sol#L105


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change variables and function calls from using USDV to VADER in the else part of the conditional which has to return the adjusted reserves when synth is not an asset i.e. an anchor and therefore base is VADER.

L144 should be changed to:
uint _adjustedReserve = iROUTER(ROUTER).getVADERAmount(reserveUSDV()) + reserveVADER();


