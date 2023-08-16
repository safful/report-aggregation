## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Malicious early user/attacker can malfunction the contract and even freeze users' funds in edge cases](https://github.com/code-423n4/2022-01-xdefi-findings/issues/156) 

# Handle

WatchPug


# Vulnerability details

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L151-L151

```solidity
    _pointsPerUnit += ((newXDEFI * _pointsMultiplier) / totalUnitsCached);
```

In the current implementation,  `_pointsPerUnit` can be changed in `updateDistribution()` which can be called by anyone. 

A malicious early user can `lock()` with only `1 wei` of XDEFI and makes `_pointsPerUnit` to be very large, causing future users not to be able to `lock()` and/or `unlock()` anymore due to overflow in arithmetic related to `_pointsMultiplier`.

As a result, the contract can be malfunctioning and even freeze users' funds in edge cases.

### PoC

Given:

- bonusMultiplierOf[30 days] = 100

1. Alice `lock()` `1 wei` of XDEFI for 30 days as the first user of the contract. Got `1` units, and `totalUnits` now is `1`;
2. Alice sends `170141183460469 wei` of `XDEFI` to the contract and calls `updateDistribution()`:

```solidity
    _pointsPerUnit += ((170141183460469 * 2**128) / 1);
```

3. Bob tries to `lock()` `1,100,000 * 1e18` of `XDEFI` for 30 days, the tx will fail, as `_pointsPerUnit * units` overlows;
4. Bob `lock()` `1,000,000 * 1e18` of `XDEFI` for 30 days;
5. The rewarder sends `250,000 * 1e18` of `XDEFI` to the contract and calls `updateDistribution()`: 

```solidity
    _pointsPerUnit += ((250_000 * 1e18 * 2**128) / (1_000_000 * 1e18 + 1));
```

6. 30 days later, Bob tries to call `unlock()`, the tx will fail, as `_pointsPerUnit * units` overflows. 


### Recomandation

Uniswap v2 solved a similar problem by sending the first 1000 lp tokens to the zero address.

The same solution should work here, i.e., on constructor set an initial amount (like 1e8) for `totalUnits`  

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L39-L44

```solidity
constructor (address XDEFI_, string memory baseURI_, uint256 zeroDurationPointBase_) ERC721("Locked XDEFI", "lXDEFI") {
        require((XDEFI = XDEFI_) != address(0), "INVALID_TOKEN");
        owner = msg.sender;
        baseURI = baseURI_;
        _zeroDurationPointBase = zeroDurationPointBase_;

        totalUnits = 100_000_000;
    }
```

