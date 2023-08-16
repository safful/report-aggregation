## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Basket.sol#handleFees()` could potentially cause disruption of minting and burning](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/79) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L110-L129

```solidity=
function handleFees() private {
    if (lastFee == 0) {
        lastFee = block.timestamp;
    } else {
        uint256 startSupply = totalSupply();

        uint256 timeDiff = (block.timestamp - lastFee);
        uint256 feePct = timeDiff * licenseFee / ONE_YEAR;
        uint256 fee = startSupply * feePct / (BASE - feePct);

        _mint(publisher, fee * (BASE - factory.ownerSplit()) / BASE);
        _mint(Ownable(address(factory)).owner(), fee * factory.ownerSplit() / BASE);
        lastFee = block.timestamp;

        uint256 newIbRatio = ibRatio * startSupply / totalSupply();
        ibRatio = newIbRatio;

        emit NewIBRatio(ibRatio);
    }
}
```

`timeDiff * licenseFee` can be greater than `ONE_YEAR` when `timeDiff` and/or `licenseFee` is large enough, which makes `feePct` to be greater than `BASE` so that `BASE - feePct` will revert on underflow.


## Impact

Minting and burning of the basket token are being disrupted until the publisher update the `licenseFee`.

## Proof of Concept

1. Create a basket with a `licenseFee` of `1e19` or 1000% per year and mint 1 basket token;
2. The basket remain inactive (not being minted or burned) for 2 months;
3. Calling `mint` and `burn` reverts at `handleFees()`.

## Recommended Mitigation Steps

Limit the max value of `feePct`.

