## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Wrong order in Approval event](https://github.com/code-423n4/2021-08-notional-findings/issues/55) 

# Handle

pauliax


# Vulnerability details

## Impact
function transferFrom in nTokenERC20Proxy emits Approval event:
  emit Approval(msg.sender, from, newAllowance);
The order of the parameters is wrong, 'msg.sender' and 'from' should be in the opposite order. This may confuse frontends or other services that consume these events from the outside.

## Recommended Mitigation Steps
  emit Approval(from, msg.sender, newAllowance);

