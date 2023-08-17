## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Reentrancy issue in `yVault.deposit`](https://github.com/code-423n4/2022-04-jpegd-findings/issues/81) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/e72861a9ccb707ced9015166fbded5c97c6991b6/contracts/vaults/yVault/yVault.sol#L144-L145


# Vulnerability details

## Impact
In `deposit`, the balance is cached and then a `token.transferFrom` is triggered which can lead to exploits if the `token` is a token that gives control to the sender, like ERC777 tokens.

#### POC
Initial state: `balance() = 1000`, shares `supply = 1000`.
Depositing 1000 amount should mint 1000 supply, but one can split the 1000 amounts into two 500 deposits and use re-entrancy to profit.

- Outer `deposit(500)`: `balanceBefore = 1000`. Control is given to attacker ...
- Inner `deposit(500)`: `balanceBefore = 1000`. `shares = (_amount * supply) / balanceBefore = 500 * 1000 / 1000 = 500` shares are minted ...
- Outer `deposit(500)` continues with the mint: `shares = (_amount * supply) / balanceBefore = 500 * 1500 / 1000 = 750` are minted.
- Withdrawing the `500 + 750 = 1250` shares via `withdraw(1250)`, the attacker receives `backingTokens = (balance() * _shares) / supply = 2000 * 1250 / 2250 = 1111.111111111`. The attacker makes a profit of `1111 - 1000 = 111` tokens.
- They repeat the attack until the vault is drained.

## Recommended Mitigation Steps
The `safeTransferFrom` should be the last call in `deposit`.


