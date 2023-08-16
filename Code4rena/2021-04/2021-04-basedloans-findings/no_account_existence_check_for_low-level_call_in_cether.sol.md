## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [No account existence check for low-level call in CEther.sol](https://github.com/code-423n4/2021-04-basedloans-findings/issues/16) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Low-level calls call/delegatecall/staticcall return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling.

The doTransferOut() function was changed from using a transfer() function (which reverts) to a call() function (which returns a boolean), however there is no account existence check for the destination address to. If it doesn’t exist, for some reason, call will still return true (not throw an exception) and successfully pass the return value check on the next line.

The checked call paths don’t seem vulnerable because they use msg.sender/admin and not a user-controlled address, but this may be a risk if used later in other contexts. Hence rating as low-risk.

For reference, see this related high-risk severity finding from Trail of Bit’s audit of Hermez Network: https://github.com/trailofbits/publications/blob/master/reviews/hermez.pdf

## Proof of Concept

https://github.com/code-423n4/2021-04-basedloans/blob/5c8bb51a3fdc334ea0a68fd069be092123212020/code/contracts/CEther.sol#L145-L148

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-calls

https://docs.soliditylang.org/en/v0.8.4/control-structures.html#error-handling-assert-require-revert-and-exceptions


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Check for account-existence before the call() to make this safely extendable to user-controlled address contexts in future.

