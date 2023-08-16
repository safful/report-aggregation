## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed
- resolved

# [TokenInitialized token parameter is always empty](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/106) 

# Handle

pauliax


# Vulnerability details

## Impact
function createToken emits TokenInitialized event, however, it does it before actually deploying the token so address(token) will always be empty (0x0):
   emit TokenInitialized(address(token), _templateId, _data);
   token = deployToken(_templateId, _integratorFeeAccount);
This may confuse external consumers of this event.

## Recommended Mitigation Steps
Usually, a good practice is to emit events in the end after all the actions are done.

