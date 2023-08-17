## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Inconsistent Order Book Accounting When Working With Transfer-On-Fee or Deflationary Tokens](https://github.com/code-423n4/2022-05-rubicon-findings/issues/112) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L557


# Vulnerability details

## Background

A transfer-on-fee token or a deflationary/rebasing token, causing the received amount to be less than the accounted amount. For instance, a deflationary tokens might charge a certain fee for every transfer() or transferFrom()

Rubicon Finance supports the trading of any ERC20 token, and anyone can liquidity pool for a new token. Thus, it is possible that such a transfer-on-fee token or a deflationary/rebasing token be used in the protocol.

Based on the source code and comment of `BathToken._deposit()`, it appears that the team is aware of this issue, and proactively implemented control (before & after balance checks) to deal with deflationary tokens.

[https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L557](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L557)

```solidity
function _deposit(uint256 assets, address receiver)
    internal
    returns (uint256 shares)
{
    uint256 _pool = underlyingBalance();
    uint256 _before = underlyingToken.balanceOf(address(this));

    // **Assume caller is depositor**
    underlyingToken.transferFrom(msg.sender, address(this), assets);
    uint256 _after = underlyingToken.balanceOf(address(this));
    assets = _after.sub(_before); // Additional check for deflationary tokens

    (totalSupply == 0) ? shares = assets : shares = (
        assets.mul(totalSupply)
    ).div(_pool);

    // Send shares to designated target
    _mint(receiver, shares);
    ..SNIP..
}
```

However, such control was not consistently applied across the protocol, and might cause the internal accounting of the orderbook to be incorrect.

## Proof-of-Concept

If the `pay_gem` token is an deflationary token, the `info.pay_amt` and the actual amount of `pay_gem` tokens received will not be in sync. 

For instance, assume that  XYZ token is a deflation token that charges 10% fee for every transfer. If an `offer(100, XYZ, 100, DAI)` is executed, an order with 100 XYZ (pay) and 100 DAI (buy) will be added to the orderbook. However, the orderbook will only received 90 XYZ, thus only 90 XYZ is ecrowed in the orderbook. This discrepancy would break the internal accounting system of the order book.

[https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconMarket.sol#L392](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/RubiconMarket.sol#L392)

```solidity
/// @notice Key function to make a new offer. Takes funds from the caller into market escrow.
function offer(
    uint256 pay_amt,
    ERC20 pay_gem,
    uint256 buy_amt,
    ERC20 buy_gem
) public virtual can_offer synchronized returns (uint256 id) {
    ..SNIP..
    OfferInfo memory info;
    info.pay_amt = pay_amt;
    info.pay_gem = pay_gem;
    info.buy_amt = buy_amt;
    info.buy_gem = buy_gem;
    info.owner = msg.sender;
    info.timestamp = uint64(block.timestamp);
    id = _next_id();
    offers[id] = info;

    require(pay_gem.transferFrom(msg.sender, address(this), pay_amt));
    ..SNIP..
}
```

## Impact

The internal accounting system of the order book would be inaccurate or break, affecting the protocol operation.

## Recommended Mitigation Steps

In the `offer` function, get the actual received amount by calculating the difference of token balance before and after the transfer, and set the `info.pay_amt` to the actual received amount.

Alternatively, the team might want to consider implementing whitelisting mechanism so that deflationary tokens will not be supported if the risk of allowing permissionless creation of pool with arbitrary token deems to be significant. A DAO may be formed in the future to manage the whitelisting.

