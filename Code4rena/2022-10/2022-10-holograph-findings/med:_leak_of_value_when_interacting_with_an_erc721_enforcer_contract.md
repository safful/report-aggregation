## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [MED: leak of value when interacting with an ERC721 enforcer contract](https://github.com/code-423n4/2022-10-holograph-findings/issues/468) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/contracts/enforcer/HolographERC721.sol#L962


# Vulnerability details

## Description

HolographERC721.sol is an enforcer of the ERC721 standard. In its fallback function, it calls the actual implementation in order to handle additional logic. 

If Holographer is called with no calldata and some msg.value, the call will reach the  receive() function, which does not forward the call down to the implementation.

This can be a serious value leak issue, because the underlying implementation may have valid behavior for handling sending of value. For example, it can mint the next available tokenID and credit it to the user. Since this logic is never reached, the entire msg.value is just leaked.

## Impact

Leak of value when interacting with an NFT using the receive() or fallback() callback. Note that if NFT implements fallback OR receive() function, execution will never reach either of them from the enforcer's receive() function.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Funnel receive() empty calls down to the implementation.