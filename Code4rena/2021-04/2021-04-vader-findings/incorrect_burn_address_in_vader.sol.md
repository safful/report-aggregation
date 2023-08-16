## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed

# [Incorrect burn address in Vader.sol](https://github.com/code-423n4/2021-04-vader-findings/issues/202) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The internal _transfer() function is called from external facing transfer(), transferFrom() and transferTo() functions all of which have different sender addresses. It is msg.sender for transfer(), sender parameter for transferFrom() and tx.origin for transferTo(). These different senders are reflected in the sender parameter of _transfer() function. While this sender parameter is correctly used for transfer of tokens within _transfer, the call to _burn() on L129 incorrectly uses msg.sender as the burn address which is correct only in the case of the transfer() caller's context. This is incorrect for transferFrom() and transferTo() caller contexts.

This will incorrectly burn the fees from a different (intermediate contract) account for all users of the protocol interacting with the transferTo() and transferFrom() functions and lead to incorrect accounting of token balances or exceptional conditions. Protocol will break and lead to fund loss.

## Proof of Concept

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L129

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L122-L134

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L91-L94

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L108-L112

https://github.com/code-423n4/2021-04-vader/blob/3041f20c920821b89d01f652867d5207d18c8703/vader-protocol/contracts/Vader.sol#L116-L119



## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change L129 to: _burn(sender, _fee);


