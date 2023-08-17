## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [[WP-H0] Fake balances can be created for not-yet-existing ERC20 tokens, which allows attackers to set traps to steal funds from future users](https://github.com/code-423n4/2022-05-cally-findings/issues/225) 

# Lines of code

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L158-L201


# Vulnerability details

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L158-L201

```solidity
function createVault(
    uint256 tokenIdOrAmount,
    address token,
    ...
) external returns (uint256 vaultId) {
    ...
    Vault memory vault = Vault({
        ...
    });

    // vault index should always be odd
    vaultIndex += 2;
    vaultId = vaultIndex;
    _vaults[vaultId] = vault;

    // give msg.sender vault token
    _mint(msg.sender, vaultId);

    emit NewVault(vaultId, msg.sender, token);

    // transfer the NFTs or ERC20s to the contract
    vault.tokenType == TokenType.ERC721
        ? ERC721(vault.token).transferFrom(msg.sender, address(this), vault.tokenIdOrAmount)
        : ERC20(vault.token).safeTransferFrom(msg.sender, address(this), vault.tokenIdOrAmount);
}
```

https://github.com/code-423n4/2022-05-cally/blob/1849f9ee12434038aa80753266ce6a2f2b082c59/contracts/src/Cally.sol#L23-L34

```solidity
import "solmate/utils/SafeTransferLib.sol";

...

contract Cally is CallyNft, ReentrancyGuard, Ownable {
    using SafeTransferLib for ERC20;
    ...
```

When creating a new vault, solmate's `SafeTransferLib` is used for pulling `vault.token` from the caller's account, this issue won't exist if OpenZeppelin's SafeERC20 is used instead.

That's because there is a subtle difference between the implementation of solmate's `SafeTransferLib` and OZ's `SafeERC20`:

OZ's `SafeERC20` checks if the token is a contract or not, solmate's `SafeTransferLib` does not.

See: https://github.com/Rari-Capital/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

> Note that none of the functions in this library check that a token has code at all! That responsibility is delegated to the caller.

As a result, when the token's address has no code, the transaction will just succeed with no error.

This attack vector was made well-known by the qBridge hack back in Jan 2022.

For our project, this alone still won't be a problem, a vault created and wrongfully accounted for a certain amount of balance for a non-existing token won't be much of a problem, there will be no fund loss as long as the token stays that way (being non-existing).

However, it's becoming popular for protocols to deploy their token across multiple networks and when they do so, a common practice is to deploy the token contract from the same deployer address and with the same nonce so that the token address can be the same for all the networks.

For example: $1INCH is using the same token address for both Ethereum and BSC; Gelato's $GEL token is using the same token address for Ethereum, Fantom and Polygon.

A sophisticated attacker can exploit it by taking advantage of that and setting traps on multiple potential tokens to steal from the future users that deposits with such tokens.

### PoC

Given:

- ProjectA has TokenA on another network;
- ProjectB has TokenB on another network;
- ProjectC has TokenC on another network;

1. The attacker `createVault()` for `TokenA`, `TokenB`, and `TokenC` with `10000e18` as `tokenIdOrAmount` each;
2. A few months later, ProjectB lunched `TokenB` on the local network at the same address;
3. Alice created a vault with `11000e18 TokenB`;
4. The attacker called `initiateWithdraw()` and then `withdraw()` to receive `10000e18 TokenB`.

In summary, one of the traps set by the attacker was activated by the deployment of  `TokenB` and Alice was the victim. As a result, `10000e18 TokenB` was stolen by the attacker.

### Recommendation

Consider using OZ's `SafeERC20` instead.

