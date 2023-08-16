## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed

# [Flash attack mitigation does not work as intended in USDV.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/138) 

# Handle

0xRajeev


# Vulnerability details

## Impact

One of the stated protocol (review) goals is to detect susceptibility to “Any attack vectors using flash loans on Anchor price, synths or lending.” As such, USDV contract aims to protect against flash attacks using flashProof() modifier which uses the following check in isMature() to determine if currently executing contract context is at least blockDelay duration ahead of the previous context: lastBlock[tx.origin] + blockDelay <= block.number

However, blockDelay state variable is not initialized which means it has a default uint value of 0. So unless it is set to >= 1 by setParams() which can be called only by the DAO (which currently does not have the capability to call setParams() function), blockDelay will be 0 which allows current executing context (block.number) to be the same as the previous one (lastBlock[tx.origin]). This effectively allows multiple calls on this contract to be executed in the same transaction of a block which enables flash attacks as opposed to what is expected as commented on L41: "// Stops an EOA doing a flash attack in same block"

Even if the DAO can call setParams() to change blockDelay to >= 1, there is a big window of opportunity for flash attacks until the DAO votes, finalises and approves such a proposal. Moreover, such proposals can be cancelled by a DAO minority or replaced by a malicious DAO minority to launch flash attacks.

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L22

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L140-L142

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L35-L44

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/USDV.sol#L174



## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Initialize blockDelay to >= 1 at declaration or in constructor.


