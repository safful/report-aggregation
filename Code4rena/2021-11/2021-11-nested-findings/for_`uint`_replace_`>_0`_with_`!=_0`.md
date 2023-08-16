## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [For `uint` replace `> 0` with `!= 0`](https://github.com/code-423n4/2021-11-nested-findings/issues/8) 

# Handle

0x0x0x


# Vulnerability details

## Proof of Concept
For unsigned integers, it is cheaper to check ` != 0` than ` > 0`. Both provide the same logic.
## Occurences
```
contracts/FeeSplitter.sol:105:        require(_accounts.length > 0 && _accounts.length == _weights.length, "FeeSplitter: ARRAY_LENGTHS_ERR");
contracts/FeeSplitter.sol:172:        require(_totalWeights > 0, "FeeSplitter: TOTAL_WEIGHTS_ZERO");
contracts/FeeSplitter.sol:263:        require(_weight > 0, "FeeSplitter: ZERO_WEIGHT");
contracts/NestedBuybacker.sol:97:        if (feeSplitter.getAmountDue(address(this), _sellToken) > 0) {
contracts/NestedFactory.sol:69:        require(i > 0, "NestedFactory::removeOperator: Cant remove non-existent operator");
contracts/NestedFactory.sol:94:        require(_orders.length > 0, "NestedFactory::create: Missing orders");
contracts/NestedFactory.sol:110:        require(_orders.length > 0, "NestedFactory::addTokens: Missing orders");
contracts/NestedFactory.sol:124:        require(_orders.length > 0, "NestedFactory::swapTokenForTokens: Missing orders");
contracts/NestedFactory.sol:143:        require(_orders.length > 0, "NestedFactory::sellTokensToNft: Missing orders");
contracts/NestedFactory.sol:163:        require(_orders.length > 0, "NestedFactory::sellTokensToWallet: Missing orders");
contracts/NestedFactory.sol:194:        require(_orders.length > 0, "NestedFactory::destroy: Missing orders");
contracts/NestedFactory.sol:333:            if (_inputTokenAmounts[i] - amountSpent > 0) {
contracts/NestedFactory.sol:467:        if (_amountToSpent - _amountSpent > 0) {
contracts/NestedRecords.sol:171:        require(_maxHoldingsCount > 0, "NestedRecords: INVALID_MAX_HOLDINGS");
contracts/operators/Flat/FlatOperator.sol:18:        require(amount > 0, "FlatOperator::commitAndRevert: Amount must be greater than zero");
contracts/operators/ZeroEx/ZeroExOperator.sol:42:        assert(amountBought > 0);
contracts/operators/ZeroEx/ZeroExOperator.sol:43:        assert(amountSold > 0);
```

