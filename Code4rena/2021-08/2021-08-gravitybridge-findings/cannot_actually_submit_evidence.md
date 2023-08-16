## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Cannot actually submit evidence](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/64) 

# Handle

jmak


# Vulnerability details

## Impact
Detailed description of the impact of this finding. The SubmitBadSignatureEvidence is not actually registered in the handler and hence no one can actually submit this message, rendering the message useless. This harms the security model of Gravity since validators have no disincentive to attempt to collude and take over the bridge. 

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.
The SubmitBadSignatureEvidence handler is omitted from module/x/gravity/handler.go

## Tools Used
Visual inspection

## Recommended Mitigation Steps
Handle the MsgSubmitBadSignatureEvidence in module/x/gravity/handler.go. 

