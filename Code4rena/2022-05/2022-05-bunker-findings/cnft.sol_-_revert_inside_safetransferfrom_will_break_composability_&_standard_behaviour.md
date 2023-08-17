## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [CNft.sol - revert inside safeTransferFrom will break composability & standard behaviour](https://github.com/code-423n4/2022-05-bunker-findings/issues/93) 

# Lines of code

https://github.com/bunkerfinance/bunker-protocol/blob/752126094691e7457d08fc62a6a5006df59bd2fe/contracts/CNft.sol#L204


# Vulnerability details

The function safeTransferFrom is a standard interface in ERC1155, and its expected to succeed if all the parametes are valid, and revert on error, which is not the case here so its a deviation.

Refer to the EIP-1155 safeTransferFrom rules:
> MUST revert if _to is the zero address.
> MUST revert if balance of holder for token _id is lower than the _value sent to the recipient.
> MUST revert on any other error.

There is no loss of assets, but the assets or tokens and CNft contract can be unusable by other protocols, and likelihood & impact of this issue is high.

## Impact
If other protocols want to integrate CNft, then in that case just for CNft Contract / tokens, they have to take exception and use safeBatchTransferFrom, instead of safeTransferFrom. If they dont take care of this exception, then their protocol functions will fail while using CNft, even if valid values are given.

## Proof of Concept
Contract : CNft.sol 
Function : safeTransferFrom

> Line 204   revert("CNFT: Use safeBatchTransferFrom instead");

## Recommended Mitigation Steps
Instead of revert, call function safeBatchTransferFrom with 1 item in the array, e.g.,
> safeBatchTransferFrom(from, to, [id], [amount], data)


