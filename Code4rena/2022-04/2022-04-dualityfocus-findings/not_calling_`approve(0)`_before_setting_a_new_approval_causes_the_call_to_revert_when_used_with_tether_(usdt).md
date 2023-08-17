## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Not calling `approve(0)` before setting a new approval causes the call to revert when used with Tether (USDT)](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/39) 

# Lines of code

https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/vault_and_oracles/FlashLoan.sol#L48
https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/vault_and_oracles/FlashLoan.sol#L58
https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/vault_and_oracles/UniV3LpVault.sol#L418


# Vulnerability details

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens.  For example Tether (USDT)'s `approve()` function will revert if the current approval is not zero, to protect against front-running changes of approvals.

## Impact
The code as currently implemented does not handle these sorts of tokens properly when they're a Uniswap pool asset, which would prevent USDT, the sixth largest pool, from being used by this project. This project relies heavily on Uniswap, so this would hamper future growth and availability of the protocol.

## Proof of Concept

1. File: contracts/vault_and_oracles/FlashLoan.sol (line [48](https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/vault_and_oracles/FlashLoan.sol#L48))
```solidity
        IERC20(assets[0]).approve(address(LP_VAULT), amounts[0]);
```

2. File: contracts/vault_and_oracles/FlashLoan.sol (line [58](https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/vault_and_oracles/FlashLoan.sol#L58))
```solidity
        IERC20(assets[0]).approve(address(LENDING_POOL), amountOwing);
```

3. File: contracts/vault_and_oracles/UniV3LpVault.sol (line [418](https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/vault_and_oracles/UniV3LpVault.sol#L418))
```solidity
            IERC20Detailed(params.asset).approve(msg.sender, owedBack);
```

There are other calls to `approve()`, but they correctly set the approval to zero after the transfer is done, so that the next approval can go through.

## Tools Used
Code inspection

## Recommended Mitigation Steps
Use OpenZeppelin’s `SafeERC20`'s `safeTransfer()` instead


