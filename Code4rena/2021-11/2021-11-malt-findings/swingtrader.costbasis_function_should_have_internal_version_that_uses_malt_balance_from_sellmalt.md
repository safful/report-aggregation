## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SwingTrader.costBasis function should have internal version that uses Malt balance from sellMalt](https://github.com/code-423n4/2021-11-malt-findings/issues/73) 

# Handle

hyh


# Vulnerability details

## Impact

ERC20 balanceOf call is costly. Malt balance is read twice in sellMalt call, which isn't needed, so gas is overspent here.

## Proof of Concept

Malt balanceOf(address(this)) is called twice:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/SwingTrader.sol#L86
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/SwingTrader.sol#L150

## Recommended Mitigation Steps

It's recommended to make internal version of costBasis that takes Malt balance as an argument.

Now:
```
uint256 totalMaltBalance = malt.balanceOf(address(this));
...
(...) = costBasis();
...
function costBasis() public view returns (uint256 cost, uint256 decimals) {
...
uint256 maltBalance = malt.balanceOf(address(this));
...
```
To be:
```
uint256 totalMaltBalance = malt.balanceOf(address(this));
...
(...) = _costBasis(totalMaltBalance);
...
function _costBasis(uint256 maltBalance) internal view returns (...) {
...
function costBasis() public view returns (...) {
	return _costBasis(malt.balanceOf(address(this)));
}
```

