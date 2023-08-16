## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed

# [Tokens can get locked and funds lost when minting is disabled in Vader.sol and USDV.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/238) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The flipMinting() function can disable/stop conversion/redeeming of VADER<>USDV tokens upon DAO approval (when that functionality is added). When minting is disabled (i.e. false), the convert functions in USDV.sol accept VADER tokens from sender (L170) but do not burn them to mint the sender the equivalent USDV tokens. When minting is disabled (i.e. false), the redeem functions in USDV.sol accept USDV tokens from sender (L188) but do not burn them to mint the sender the equivalent VADER tokens. Both paths silently return 0 without reverting the transaction thus trapping the sent tokens and leaving the users with lost funds. Protocol will break and funds will be lost.

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L171-L177

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L174-L181

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L165-L172

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L183-L191

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L238-L243


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Revert in the paths (instead of silently returning) when minting is disabled so that tokens are not accepted for conversion or redemption.

