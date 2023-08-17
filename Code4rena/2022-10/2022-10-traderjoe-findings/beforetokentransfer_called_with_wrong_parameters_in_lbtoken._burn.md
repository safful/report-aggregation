## Tags

- bug
- 2 (Med Risk)
- primary issue
- sponsor confirmed
- selected for report
- M-02

# [beforeTokenTransfer called with wrong parameters in LBToken._burn](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/108) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/37258d595d596c195507234f795fa34e319b0a68/src/LBToken.sol#L237


# Vulnerability details

## Impact
In `LBToken._burn`, the `_beforeTokenTransfer` hook is called with `from = address(0)` and `to = _account`:
```solidity
_beforeTokenTransfer(address(0), _account, _id, _amount);
```
Through a lucky coincidence, it turns out that this in the current setup does not cause a high severity issue. `_burn` is always called with `_account = address(this)`, which means that `LBPair._beforeTokenTransfer` is a NOP. However, this wrong call is very dangerous for future extensions or protocol that built on top of the protocol / fork it.

## Proof Of Concept
Let's say the protocol is extended with some logic that needs to track mints / burns. The canonical way to do this would be:
```solidity
function _beforeTokenTransfer(
        address _from,
        address _to,
        uint256 _id,
        uint256 _amount
    ) internal override(LBToken) {
	if (_from == address(0)) {
		// Mint Logic
	} else if (_to == address(0)) {
		// Burn Logic
	}
}
```
Such an extension would break, which could lead to loss of funds or a bricked system.

## Recommended Mitigation Steps
Call the hook correctly:
```solidity
_beforeTokenTransfer(_account, address(0), _id, _amount);
```