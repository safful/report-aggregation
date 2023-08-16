## Tags

- bug
- question
- 2 (Med Risk)
- sponsor confirmed

# [_totalSupply not updated in _transferMint() and _transferBurn() ](https://github.com/code-423n4/2021-11-overlay-findings/issues/59) 

# Handle

gpersoon


# Vulnerability details

## Impact
The functions _transferMint() and _transferBurn() of OverlayToken.sol don't update _totalSupply.
Whereas the similar functions _mint() and _burn() do update _totalSupply.

This means that _totalSupply and totalSupply() will not show a realistic view of the total OVL tokens.

For the protocol itself it isn't such a problem because this value isn't used in the protocol (as far as I can see).
But other protocols building on Overlay may use it, as well as user interfaces and analytic platforms.

## Proof of Concept
https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/ovl/OverlayToken.sol#L349-L364
```JS
function _mint( address account, uint256 amount) internal virtual {
   ...
      _totalSupply += amount;

https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/ovl/OverlayToken.sol#L376-L395
```JS
function _burn(address account, uint256 amount) internal virtual {
   ...
        _totalSupply -= amount;

https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/ovl/OverlayToken.sol#L194-L212

https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/ovl/OverlayToken.sol#L268-L286

## Tools Used

## Recommended Mitigation Steps
Update _totalSupply  in _transferMint() and _transferBurn() 

