## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Safe addresses can only be added but not removed](https://github.com/code-423n4/2021-06-gro-findings/issues/51) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The addSafeAddress()  takes an address and adds it to a “safe list". This is used in eoaOnly() to give exemption to safe addresses that are trusted smart contracts, when all other smart contacts are prevented from protocol interaction. The stated purpose is to allow only such partner/trusted smart contract integrations (project rep mentioned Argent wallet as the only one for now but that may change) an exemption from potential flash loan threats. But if there a safe listed integration that needs to be later disabled, it cannot be done. The protocol will have to rely on other measures (outside the scope of this contest) to prevent flash loan manipulations which are specified as an area of critical concern.

Scenario: A trusted integration/partner address is added to safe list. But that wallet/protocol/DApp is later manipulated (by project, its users or an attacker) to somehow launch a flash loan attack on the protocol. However, its address cannot be removed from the safe list and the protocol cannot prevent flash loan manipulations from that source because of its exemption. Contract/project will have to be redeployed.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L171-L174

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L176-L178

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L266-L272


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change addSafeAddress() to isSafeAddress() with an additional bool parameter to allow both enabling/disabling of safe addresses.

