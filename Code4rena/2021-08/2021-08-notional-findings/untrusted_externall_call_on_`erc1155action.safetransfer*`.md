## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Untrusted externall call on `ERC1155Action.safeTransfer*`](https://github.com/code-423n4/2021-08-notional-findings/issues/62) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `ERC1155Action.safeTransferFrom` / `ERC1155Action.safeBatchTransferFrom` functions do not follow the [recommended re-entrancy protection guidelines](https://docs.soliditylang.org/en/v0.8.0/security-considerations.html#use-the-checks-effects-interactions-pattern) and allow a re-entrancy through the `onERC1155Received` while state still has to be written.

## Impact
The re-entrancy doesn't seem to open any attacks currently as the re-entrancy call happens right at the beginning and no interesting variables are set yet.

## Recommended Mitigation Steps
While no immediate re-entrancy issues could be found, it's better to add these checks, especially, as calling this function from another Notional finance function in the future might lead to unintended issues.

