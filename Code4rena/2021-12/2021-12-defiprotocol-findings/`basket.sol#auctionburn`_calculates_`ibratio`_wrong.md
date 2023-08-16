## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`Basket.sol#auctionBurn` calculates `ibRatio` wrong](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/144) 

# Handle

0x0x0x


# Vulnerability details

The function is implemented as follows:

```
function auctionBurn(uint256 amount) onlyAuction nonReentrant external override {
        uint256 startSupply = totalSupply();
        handleFees(startSupply);
        _burn(msg.sender, amount);

        uint256 newIbRatio = ibRatio * startSupply / (startSupply - amount);
        ibRatio = newIbRatio;

        emit NewIBRatio(newIbRatio);
        emit Burned(msg.sender, amount);
    }
```

When `handleFees` is called, `totalSupply` and `ibRatio` changes accordingly, but for `newIbRatio` calculation tokens minted in `handleFees` is not included. Therefore, `ibRatio` is calculated higher than it should be. This is dangerous, since last withdrawing user(s) lose their funds with this operation. In case this miscalculation happens more than once, `newIbRatio` will increase the miscalculation even faster and can result in serious amount of funds missing. At each time `auctionBurn` is called, at least 1 day (auction duration) of fees result in this miscalculation. Furthermore, all critical logic of this contract is based on `ibRatio`, this behaviour can create serious miscalculations. 

## Mitigation step

Rather than

`uint256 newIbRatio = ibRatio * startSupply / (startSupply - amount);`

A practical solution to this problem is calculating `newIbRatio` as follows:

```
uint256 supply = totalSupply();
uint256 newIbRatio = ibRatio * (supply + amount) / supply;
```

