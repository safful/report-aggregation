## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Key transfer will destroy key if from==to](https://github.com/code-423n4/2021-11-unlock-findings/issues/87) 

# Handle

kenzo


# Vulnerability details

If calling `transferFrom` with `_from == _recipient`, the key will get destroyed (meaning the key will be set as expired and set the owner's key to be 0).

## Impact
A key manager or approved might accidently destroy user's token.

Note: this requires user error and so I'm not sure if this is a valid finding.
However, few things make me think that it is valid:
- Unlock protocol checks for transfer to 0-address, so some input validation is there
- Since other entities other than the owner can be allowed to transfer owner's token, it might be best to make sure such accidental mistake could not happen.
- This scenario manifests a unique and probably unintended behavior


## Proof of Concept
By following `transferFrom`'s execution:
https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinTransfer.sol#L109:#L166
One can see that in the case where `_from == _recipient` with a valid key:
- The function will deduct transfer fee from the key
- The function will incorrectly add more time to the key's expiration ([L151](https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinTransfer.sol#L151))
- The function will expire and reset the key ([L155](https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinTransfer.sol#L155:#L158))
Therefore, the user will lose his key without getting a refund.

## Recommended Mitigation Steps
Add a require statement in the beginning of `transferFrom`:
`require(_from != _recipient, 'TRANSFER_TO_SELF');`

