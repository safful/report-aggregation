## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Migration can permanently fail if user specifies different lengths for `selectors` and `plugins`](https://github.com/code-423n4/2022-07-fractional-findings/issues/115) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/Vault.sol#L73-L82
https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Migration.sol#L72-L99
https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/VaultRegistry.sol#L174


# Vulnerability details

## Impact
In `propose()` in Migration.sol, there is no check that the lengths of the `selectors` and `plugins` arrays are the same. This means that if a migration is successful, the `install()` function in Vault.sol could revert beacuse we access an array out of bounds. This prevents a new vault being created thereby permanently locking assets inside the vault.

## Proof of Concept
1. user starts a new migration proposal where `selectors.length != plugins.length`
2. enough users join proposal and the buyout bid starts
3. buyout bid is successful and migration starts with `settleVault()`
4. a new vault is cloned with `create()` -> `registry.deployFor()` -> `vault.install(selectors, plugins)`
5. a. If `selectors.length > plugins.length` then we get an out of bounds error and transaction reverts
    b. If `selectors.length < plugins.length` then the excess values in `plugins` is ignored which is tolerable
6. In scenario a., the migration fails and a new migration cannot start so assets in the vault are permanently locked

This may seem quite circumstantial as this problem only occurs if a user specifies `selectors` and `plugins` wrongly however it is very easy for an attacker to perform this maliciously with no cost on their behalf, it is highly unlikely that users will be able to spot a malicious migration.

## Tools Used
VS Code
## Recommended Mitigation Steps
Consider adding a check in `propose()` to make sure that the lengths match i.e.
```solidity
function propose(
        address _vault,
        address[] calldata _modules,
        address[] calldata _plugins,
        bytes4[] calldata _selectors,
        uint256 _newFractionSupply,
        uint256 _targetPrice
    ) external {
        // @Audit Make sure that selectors and plugins match
        require(_selectors.length == _plugins.length, "Plugin lengths do not match");
        // Reverts if address is not a registered vault
        (, uint256 id) = IVaultRegistry(registry).vaultToToken(_vault);
        if (id == 0) revert NotVault(_vault);
        // Reverts if buyout state is not inactive
        (, , State current, , , ) = IBuyout(buyout).buyoutInfo(_vault);
        State required = State.INACTIVE;
        if (current != required) revert IBuyout.InvalidState(required, current);

        // Initializes migration proposal info
        Proposal storage proposal = migrationInfo[_vault][++nextId];
        proposal.startTime = block.timestamp;
        proposal.targetPrice = _targetPrice;
        proposal.modules = _modules;
        proposal.plugins = _plugins;
        proposal.selectors = _selectors;
        proposal.oldFractionSupply = IVaultRegistry(registry).totalSupply(
            _vault
        );
        proposal.newFractionSupply = _newFractionSupply;
    }
```

Additionally, I would suggest adding such a check in the `install()` function as this may prevent similiar problems if new modules are added

