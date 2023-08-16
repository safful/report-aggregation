## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [`MainToken.set_minter()` emits `SetMinter` events when the minter hasn't changed](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/62) 

# Handle

pants


# Vulnerability details

The function `MainToken.set_minter()` emits `SetMinter` events when the minter hasn't changed and left as it was before that transaction.

## Impact
There is no reason to emit these `SetMinter` events because nothing has changed in the system. Such events are only going to confuse users.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Emit these events only when the new minter is different than the old one.

