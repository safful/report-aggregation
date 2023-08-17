## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Illuminate PT redeeming allows for burning from other accounts](https://github.com/code-423n4/2022-06-illuminate-findings/issues/387) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L114-L128


# Vulnerability details

Illuminate PT burns shares from a user supplied address account instead of user's account. With such a discrepancy a malicious user can burn all other's user shares by having the necessary shares on her balance, while burning them from everyone else.

Setting the severity to be high as this allows for system-wide stealing of user's funds.

## Proof of Concept

Redeemer's Illuminate redeem() checks the balance of msg.sender, but burns from the balance of user supplied `o` address:

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L114-L128

L120:

```solidity
uint256 amount = token.balanceOf(msg.sender);
```

L126:

```solidity
token.burn(o, amount);
```

```solidity
        address principal = IMarketPlace(marketPlace).markets(u, m, p);

        if (p == uint8(MarketPlace.Principals.Illuminate)) {
            // Get Illuminate's principal token
            IERC5095 token = IERC5095(principal);
            // Get the amount of tokens to be redeemed from the sender
            uint256 amount = token.balanceOf(msg.sender);
            // Make sure the market has matured
            if (block.timestamp < token.maturity()) {
                revert Invalid('not matured');
            }
            // Burn the prinicipal token from Illuminate
            token.burn(o, amount);
            // Transfer the original underlying token back to the user
            Safe.transferFrom(IERC20(u), lender, address(this), amount);
```

`o` address isn't validated and used as provided.

Burning proceeds as usual, Illuminate PT burns second argument `a` from the first argument `f`, i.e. `f`'s balance to be reduced by `a`:

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/marketplace/ERC5095.sol#L121-L127

```solidity
    /// @param f Address to burn from
    /// @param a Amount to burn
    /// @return bool true if successful
    function burn(address f, uint256 a) external onlyAdmin(redeemer) returns (bool) {
        _burn(f, a);
        return true;
    }
```

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/marketplace/ERC5095.sol#L7

```solidity
contract ERC5095 is ERC20Permit, IERC5095 {
```

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/marketplace/ERC20.sol#L187-L196

```solidity
    function _burn(address src, uint wad) internal virtual returns (bool) {
        unchecked {
            require(_balanceOf[src] >= wad, "ERC20: Insufficient balance");
            _balanceOf[src] = _balanceOf[src] - wad;
            _totalSupply = _totalSupply - wad;
            emit Transfer(src, address(0), wad);
        }

        return true;
    }
```

This way a malicious user owning some Illuminate PT can burn the same amount of PT as she owns from any another account, that is essentially from all other accounts, obtaining all the underlying tokens from the system. The behavior is somewhat similar to the public burn case.

## Recommended Mitigation Steps

`o` address looks to be not needed in Illuminate PT case.

Consider burning the shares from `msg.sender`, for example:

https://github.com/code-423n4/2022-06-illuminate/blob/912be2a90ded4a557f121fe565d12ec48d0c4684/redeemer/Redeemer.sol#L125-L126

```solidity
            // Burn the prinicipal token from Illuminate
-           token.burn(o, amount);
+           token.burn(msg.sender, amount);
```

