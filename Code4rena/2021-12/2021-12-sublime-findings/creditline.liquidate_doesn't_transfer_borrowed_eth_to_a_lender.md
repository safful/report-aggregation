## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [CreditLine.liquidate doesn't transfer borrowed ETH to a lender](https://github.com/code-423n4/2021-12-sublime-findings/issues/90) 

# Handle

hyh


# Vulnerability details

## Impact

Funds that are acquired from a liquidator and should be sent to a lender are left with the contract instead. The funds aren't lost, but after the fact mitigation will require manual accounting and fund transfer for each CreditLine.liquidate usage.

## Proof of Concept

ETH sent to CreditLine.liquidate by an external liquidator when `autoLiquidation` is enabled remain with the contract and aren't transferred to the lender:
https://github.com/code-423n4/2021-12-sublime/blob/main/contracts/CreditLine/CreditLine.sol#L1015

## Recommended Mitigation Steps

Add transfer to a lender for ETH case:

Now:
```
if (_borrowAsset == address(0)) {
		uint256 _returnETH = msg.value.sub(_borrowTokens, 'Insufficient ETH to liquidate');
		if (_returnETH != 0) {
				(bool success, ) = msg.sender.call{value: _returnETH}('');
				require(success, 'Transfer fail');
		}
}
```

To be:
```
if (_borrowAsset == address(0)) {
		uint256 _returnETH = msg.value.sub(_borrowTokens, 'Insufficient ETH to liquidate');
		
		(bool success, ) = _lender.call{value: _borrowTokens}('');
		require(success, 'liquidate: Transfer failed');
		
		if (_returnETH != 0) {
				(success, ) = msg.sender.call{value: _returnETH}('');
				require(success, 'liquidate: Return transfer failed');
		}
}
```

