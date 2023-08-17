## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`RubiconMarket` buys can not be disabled if offer matching is disabled](https://github.com/code-423n4/2022-05-rubicon-findings/issues/317) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconMarket.sol#L962


# Vulnerability details

## Impact

In the `RubiconMarket` contract, buys can be disabled with `setBuyEnabled`. However, if `matchingEnabled` is set to `false`, buys can not be disabled as the `require` check is located in the `_buys` function instead of checking `buyEnabled` in the `buy` function.

## Proof of Concept

[RubiconMarket.sol#L962](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconMarket.sol#L962)

```solidity
function _buys(uint256 id, uint256 amount) internal returns (bool) {
    require(buyEnabled); // @audit-info Buys can not be disabled if offer matching is disabled - this require statement should be moved to `buy` function
    if (amount == offers[id].pay_amt) {
        if (isOfferSorted(id)) {
            //offers[id] must be removed from sorted list because all of it is bought
            _unsort(id);
        } else {
            _hide(id);
        }
    }

    require(super.buy(id, amount));

    // If offer has become dust during buy, we cancel it
    if (
        isActive(id) &&
        offers[id].pay_amt < _dust[address(offers[id].pay_gem)]
    ) {
        dustId = id; //enable current msg.sender to call cancel(id)
        cancel(id);
    }
    return true;
}
```

## Tools Used

Manual review

## Recommended mitigation steps

Move the `require` check for `buyEnabled` to the `buy` function [here](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconMarket.sol#L662).

