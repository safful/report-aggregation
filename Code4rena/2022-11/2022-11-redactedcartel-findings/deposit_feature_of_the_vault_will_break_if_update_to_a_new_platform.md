## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- selected for report
- sponsor confirmed
- edited-by-warden
- M-07

# [Deposit Feature Of The Vault Will Break If Update To A New Platform](https://github.com/code-423n4/2022-11-redactedcartel-findings/issues/182) 

# Lines of code

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L73
https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L152


# Vulnerability details

## Proof of Concept

During initialization, the `AutoPxGMX` vault will grant max allowance to the platform (PirexGMX) to spend its GMX tokens in Line 97 of the constructor method below. This is required because the vault needs to deposit GMX tokens to the platform (PirexGMX) contract. During deposit, the platform (PirexGMX) contract will pull the GMX tokens within the vault and send them to GMX protocol for staking. Otherwise, the deposit feature within the vault will not work.

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L73

```solidity
File: AutoPxGmx.sol
73:     constructor(
74:         address _gmxBaseReward,
75:         address _gmx,
76:         address _asset,
77:         string memory _name,
78:         string memory _symbol,
79:         address _platform,
80:         address _rewardsModule
81:     ) Owned(msg.sender) PirexERC4626(ERC20(_asset), _name, _symbol) {
82:         if (_gmxBaseReward == address(0)) revert ZeroAddress();
83:         if (_gmx == address(0)) revert ZeroAddress();
84:         if (_asset == address(0)) revert ZeroAddress();
85:         if (bytes(_name).length == 0) revert InvalidAssetParam();
86:         if (bytes(_symbol).length == 0) revert InvalidAssetParam();
87:         if (_platform == address(0)) revert ZeroAddress();
88:         if (_rewardsModule == address(0)) revert ZeroAddress();
89: 
90:         gmxBaseReward = ERC20(_gmxBaseReward);
91:         gmx = ERC20(_gmx);
92:         platform = _platform;
93:         rewardsModule = _rewardsModule;
94: 
95:         // Approve the Uniswap V3 router to manage our base reward (inbound swap token)
96:         gmxBaseReward.safeApprove(address(SWAP_ROUTER), type(uint256).max);
97:         gmx.safeApprove(_platform, type(uint256).max);
98:     }
```

However, when the owner calls the `AutoPxGmx.setPlatform` function to update the `platform` to a new address, it does not grant any allowance to the new platform address. As a result, the new platform (PirexGMX) will not be able to pull the GMX tokens from the vault. Thus, the deposit feature of the vault will break, and no one will be able to deposit.

https://github.com/code-423n4/2022-11-redactedcartel/blob/03b71a8d395c02324cb9fdaf92401357da5b19d1/src/vaults/AutoPxGmx.sol#L152

```solidity
File: AutoPxGmx.sol
152:     function setPlatform(address _platform) external onlyOwner {
153:         if (_platform == address(0)) revert ZeroAddress();
154: 
155:         platform = _platform;
156: 
157:         emit PlatformUpdated(_platform);
158:     }
```

## Impact

The deposit feature of the vault will break, and no one will be able to deposit.

## Recommended Mitigation Steps

Ensure that allowance is given to the new platform address so that it can pull the GMX tokens from the vault.

```diff
function setPlatform(address _platform) external onlyOwner {
    if (_platform == address(0)) revert ZeroAddress();
+   if (_platform == platform) revert SamePlatformAddress();
    
+   gmx.safeApprove(platform, 0); // set the old platform approval amount to zero
+   gmx.safeApprove(_platform, type(uint256).max); // approve the new platform contract address allowance to the max

    platform = _platform;

    emit PlatformUpdated(_platform);
}
```