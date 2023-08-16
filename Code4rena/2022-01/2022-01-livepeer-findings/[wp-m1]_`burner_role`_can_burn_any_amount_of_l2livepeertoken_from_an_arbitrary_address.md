## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [[WP-M1] `BURNER_ROLE` can burn any amount of L2LivepeerToken from an arbitrary address](https://github.com/code-423n4/2022-01-livepeer-findings/issues/194) 

# Handle

WatchPug


# Vulnerability details

https://github.com/livepeer/arbitrum-lpt-bridge/blob/ebf68d11879c2798c5ec0735411b08d0bea4f287/contracts/L2/token/LivepeerToken.sol#L36-L43

```solidity
function burn(address _from, uint256 _amount)
    external
    override
    onlyRole(BURNER_ROLE)
{
    _burn(_from, _amount);
    emit Burn(_from, _amount);
}
```

Using the `burn()` function of `L2LivepeerToken`, an address with `BURNER_ROLE` can burn an arbitrary amount of tokens from any address.

We believe this is unnecessary and poses a serious centralization risk.

A malicious or compromised `BURNER_ROLE` address can take advantage of this, burn the balance of a Uniswap pool and effectively steal almost all the funds from the liquidity pool (eg, Uniswap LPT-WETH Pool).

### Recommendation

Consider removing the `BURNER_ROLE` and change `burn()` function to:

```solidity
function burn(uint256 _amount)
    external
    override
{
    _burn(msg.sender, _amount);
    emit Burn(msg.sender, _amount);
}
```

https://github.com/livepeer/arbitrum-lpt-bridge/blob/49cf5401b0514511675d781a1e29d6b0325cfe88/contracts/L2/gateway/L2LPTGateway.sol#L34-L45

`Mintable(l2Lpt).burn(from, _amount);` in `L2LPTGateway.sol#outboundTransfer()` should also be replaced with:

```solidity
Mintable(l2Lpt).transferFrom(from, _amount);
Mintable(l2Lpt).burn(_amount);
```

