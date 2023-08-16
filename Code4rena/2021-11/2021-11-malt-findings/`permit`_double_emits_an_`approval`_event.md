## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`permit` Double Emits An `Approval` Event](https://github.com/code-423n4/2021-11-malt-findings/issues/230) 

# Handle

leastwood


# Vulnerability details

## Impact

The `permit` function is intended to facilitate approvals though signature verification. This helps to merge the two-step token transfer process consisting of an initial token approval and subsequent transfer. The `permit` function emits an `Approval` event, however, the `_approve` function also emits the same `Approval` event. As a result, off-chain scripts monitoring the blockchain for such events will see the same event emitted twice which may cause unintended issues.

## Proof of Concept

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/ERC20Permit.sol#L58-L59
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L314

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider not emitting an `Approval` event in `permit`.

