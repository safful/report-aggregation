## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [`BathToken.sol#_deposit()` attacker can mint more shares with re-entrancy from hookable tokens](https://github.com/code-423n4/2022-05-rubicon-findings/issues/350) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L557-L568


# Vulnerability details

`BathToken.sol#_deposit()` calculates the actual transferred amount by comparing the before and after balance, however, since there is no reentrancy guard on this function, there is a risk of re-entrancy attack to mint more shares.

Some token standards, such as ERC777, allow a callback to the source of the funds (the `from` address) before the balances are updated in `transferFrom()`. This callback could be used to re-enter the function and inflate the amount.

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L557-L568

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
    ...
```

### PoC

With a ERC777 token by using the ERC777TokensSender `tokensToSend` hook to re-enter the `deposit()` function.

Given: 

-   `underlyingBalance()`: `100_000e18 XYZ`.
-   `totalSupply`: `1e18`

The attacker can create a contracts with `tokensToSend()` function, then:

1.   `deposit(1)`
    -   preBalance  = `100_000e18`;
    -   `underlyingToken.transferFrom(msg.sender, address(this), 1)`
2. reenter using `tokensToSend` hook for the 2nd call: `deposit(1_000e18)`
    -   preBalance  = `100_000e18`;
    -   `underlyingToken.transferFrom(msg.sender, address(this), 1_000e18)`
    -   postBalance = `101_000e18`;
    -   assets (actualDepositAmount) = `101_000e18 - 100_000e18 = 1_000e18`;
    -   mint `1000` shares;
3. continue with the first `deposit()` call:
    -   `underlyingToken.transferFrom(msg.sender, address(this), 1)`
    -   postBalance = `101_000e18 + 1`;
    -   assets (actualDepositAmount) = `(101_000e18 + 1) - 100_000e18 = 1_000e18 + 1`;
    -   mint `1000` shares;

As a result, with only `1 + 1_000e18` transferred to the contract, the attacker minted `2_000e18 XYZ` worth of shares.

### Recommendation

Consider adding `nonReentrant` modifier from OZ's `ReentrancyGuard`.


