## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Yeti token rebase checks the additional token amount incorrectly](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/121) 

# Handle

hyh


# Vulnerability details

# Impact

The condition isn't checked now as the whole balance is used instead of the Yeti tokens bought back from the market.
As it's not checked, the amount added to `effectiveYetiTokenBalance` during rebase can exceed the actual amount of the Yeti tokens owned by the contract.
As the before check amount is calculated as the contract net worth, it can be fixed by immediate buy back, but it will not be the case.

The deficit of Yeti tokens can materialize in net worth terms as well if Yeti tokens price will raise compared to the last used one.
In this case users will be cumulatively accounted with the amount of tokens that cannot be actually withdrawn from the contract, as its net holdings will be less then total users’ claims.
In other words, the contract will be in default if enough users claim after that.

## Proof of Concept

Now the whole balance amount is used instead of the amount bought back from market.

Rebasing amount is added to `effectiveYetiTokenBalance`, so it should be limited by extra Yeti tokens, not the whole balance:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/YETI/sYETIToken.sol#L247

## Recommended Mitigation Steps

It looks like only extra tokens should be used for the check, i.e. `yetiToken.balance - effectiveYetiTokenBalance`.

Now:
```
function rebase() external {
		...
		uint256 yetiTokenBalance = yetiToken.balanceOf(address(this));
		uint256 valueOfContract = _getValueOfContract(yetiTokenBalance);
		uint256 additionalYetiTokenBalance = ...
		if (yetiTokenBalance < additionalYetiTokenBalance) {
				additionalYetiTokenBalance = yetiTokenBalance;
		}
		effectiveYetiTokenBalance = effectiveYetiTokenBalance.add(additionalYetiTokenBalance);
...
function _getValueOfContract(uint _yetiTokenBalance) internal view returns (uint256) {
		uint256 adjustedYetiTokenBalance = _yetiTokenBalance.sub(effectiveYetiTokenBalance);
		uint256 yusdTokenBalance = yusdToken.balanceOf(address(this));
		return div(lastBuybackPrice.mul(adjustedYetiTokenBalance), (1e18)).add(yusdTokenBalance);
}
```

As the `_getValueOfContract` function isn't used elsewhere, the logic can be simplified.
To be:
```
function rebase() external {
		...
		uint256 adjustedYetiTokenBalance = (yetiToken.balanceOf(address(this))).sub(effectiveYetiTokenBalance);
		uint256 valueOfContract = _getValueOfContract(adjustedYetiTokenBalance);
		uint256 additionalYetiTokenBalance = ...
		if (additionalYetiTokenBalance > adjustedYetiTokenBalance) {
				additionalYetiTokenBalance = adjustedYetiTokenBalance;
		}
		effectiveYetiTokenBalance = effectiveYetiTokenBalance.add(additionalYetiTokenBalance);
...
function _getValueOfContract(uint _adjustedYetiTokenBalance) internal view returns (uint256) {
		uint256 yusdTokenBalance = yusdToken.balanceOf(address(this));
		return div(lastBuybackPrice.mul(_adjustedYetiTokenBalance), (1e18)).add(yusdTokenBalance);
}
```


