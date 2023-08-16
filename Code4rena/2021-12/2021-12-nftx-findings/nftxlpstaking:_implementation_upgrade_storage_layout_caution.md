## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [NFTXLPStaking: Implementation Upgrade Storage Layout Caution](https://github.com/code-423n4/2021-12-nftx-findings/issues/220) 

# Handle

GreyArt


# Vulnerability details

## Impact

From what we understand, the contracts upgrade will be performed in place, where the relevant current proxies will be pointing to the new implementations. An important restriction when doing so is that the order of which the contract state variables are declared, and their types **must be preserved.** More information can be found in [OpenZeppelin’s documentation](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#modifying-your-contracts).

For the NFTXLPStaking contract, the [version of the May contest review](https://github.com/code-423n4/2021-05-nftx/blob/main/nftx-protocol-v2/contracts/solidity/NFTXLPStaking.sol) was:

```jsx
contract NFTXLPStaking is OwnableUpgradeable {
    using SafeERC20Upgradeable for IERC20Upgradeable;

    INFTXVaultFactory public nftxVaultFactory;
    INFTXFeeDistributor public feeDistributor;
    RewardDistributionTokenUpgradeable public rewardDistTokenImpl;
    StakingTokenProvider public stakingTokenProvider;

    event PoolCreated(uint256 vaultId, address pool);
    event PoolUpdated(uint256 vaultId, address pool);
    event FeesReceived(uint256 vaultId, uint256 amount);

    struct StakingPool {
        address stakingToken;
        address rewardToken;
    }
    mapping(uint256 => StakingPool) public vaultStakingInfo;

    function __NFTXLPStaking__init(address _stakingTokenProvider) external initializer {
...
```

while the new version is

```jsx
contract NFTXLPStaking is PausableUpgradeable {
    using SafeERC20Upgradeable for IERC20Upgradeable;

    INFTXVaultFactory public nftxVaultFactory;
    IRewardDistributionToken public rewardDistTokenImpl;
    StakingTokenProvider public stakingTokenProvider;

    event PoolCreated(uint256 vaultId, address pool);
    event PoolUpdated(uint256 vaultId, address pool);
    event FeesReceived(uint256 vaultId, uint256 amount);

    struct StakingPool {
        address stakingToken;
        address rewardToken;
    }
    mapping(uint256 => StakingPool) public vaultStakingInfo;

    TimelockRewardDistributionTokenImpl public newTimelockRewardDistTokenImpl;

    function __NFTXLPStaking__init(address _stakingTokenProvider) external initializer {
...
```

Note that the `feeDistributor` has been removed. Also note that a new base contract has been added (`PausableUpgradeable` which inherits `OwnableUpgradeable`), which has 2 mappings `isGuardian` and `isPaused`. 

We however note that the current `NFTXLPStaking` implementation at [`https://etherscan.io/address/0xa64c2f3f965f055e51482bf0960ebb5f2904bf68#code`](https://etherscan.io/address/0xa64c2f3f965f055e51482bf0960ebb5f2904bf68#code) is a more recent version than that of the previous contest review. There is no change in the storage layout between this deployed version against the one being reviewed.

The ordering of state variables is determined by the C3-linearized order of contracts, so there does not seem to have been any storage collision with the change from `OwnableUpgradeable` to `PausableUpgradeable`. It also appears that the public variables are returning expected values.

## Recommended Mitigation Steps

Upgrading implementations are a tricky affair. It is highly recommended to use tools like OpenZeppelin’s upgrade plugins that validate that the new implementation is upgrade safe and is compatible with the previous one.

