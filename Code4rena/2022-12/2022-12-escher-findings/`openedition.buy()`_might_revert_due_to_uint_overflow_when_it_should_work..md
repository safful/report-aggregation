## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-03

# [`OpenEdition.buy()` might revert due to uint overflow when it should work.](https://github.com/code-423n4/2022-12-escher-findings/issues/175) 

# Lines of code

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/OpenEdition.sol#L63


# Vulnerability details

## Impact
`OpenEdition.buy()` might revert due to uint overflow when it should work.

## Proof of Concept
`OpenEdition.buy()` validates the total funds like below.

```solidity
    function buy(uint256 _amount) external payable {
        uint24 amount = uint24(_amount);
        Sale memory temp = sale;
        IEscher721 nft = IEscher721(temp.edition);
        require(block.timestamp >= temp.startTime, "TOO SOON");
        require(block.timestamp < temp.endTime, "TOO LATE");
        require(amount * sale.price == msg.value, "WRONG PRICE"); //@audit overflow
```

Here, `amount` was declared as `uint24` and `sale.price` is `uint72`.

And it will revert when `amount * sale.price >= type(uint72).max` and such cases would be likely to happen e.g. `amount = 64(so 2^6), sale.price = 73 * 10^18(so 2^66)`.

As a result, `buy()` might revert when it should work properly.

## Tools Used
Manual Review

## Recommended Mitigation Steps
We should modify like below.

```solidity
    require(uint256(amount) * sale.price == msg.value, "WRONG PRICE");
```