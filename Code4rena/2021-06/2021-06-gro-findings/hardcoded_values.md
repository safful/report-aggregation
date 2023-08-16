## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [hardcoded values](https://github.com/code-423n4/2021-06-gro-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
There are several hardcodes values that could very well be replaced with constants.
For example:
- 10**18
- 5E17
- 10000 
- 10**4
- 3 (N_COINS)
This will make the code more readable and easier to maintain

## Proof of Concept
//https://github.com/code-423n4/2021-06-gro/blob/main/contracts/DepositHandler.sol#L206
 function roughUsd(uint256[N_COINS] memory inAmounts) private view returns (uint256 usdAmount) {
     ..
           usdAmount = usdAmount.add(inAmounts[i].mul(10**18).div(getDecimal(i)));

// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/tokens/GToken.sol#L24
abstract contract GToken is GERC20, Constants, Whitelist, IToken {
    uint256 public constant BASE = DEFAULT_DECIMALS_FACTOR;

 function applyFactor(
  ....
     if (diff >= 5E17) {

// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/vaults/yearnv2/v032/VaultAdaptorYearnV2_032.sol#L107
function updateStrategiesDebtRatio(uint256[] memory ratios) internal override {
        ..
        require(ratioTotal <= 10**4, "The total of ratios is more than 10000");

// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/Controller.sol#L317
 function emergency(uint256 coin) external onlyWhitelist {
...
       percent = 10000;

// https://github.com/code-423n4/2021-06-gro/blob/main/contracts/insurance/Insurance.sol#L144
 function getVaultDeltaForDeposit(uint256 amount)
....
  investDelta[vaultIndexes[0]] = 10000;


.\common\StructDefinitions.sol:    uint256[3] vaultCurrentAssets;
.\common\StructDefinitions.sol:    uint256[3] vaultCurrentAssetsUsd;
.\common\StructDefinitions.sol:    uint256[3] stablePercents;
.\common\StructDefinitions.sol:    uint256[3] stablecoinExposure;
.\common\StructDefinitions.sol:    uint256[3] protocolWithdrawalUsd;
.\common\StructDefinitions.sol:    uint256[3] swapInAmounts;
.\common\StructDefinitions.sol:    uint256[3] swapInAmountsUsd;
.\common\StructDefinitions.sol:    uint256[3] swapOutPercents;
.\common\StructDefinitions.sol:    uint256[3] vaultsTargetUsd;
.\interfaces\IBuoy.sol:    function stableToUsd(uint256[3] calldata inAmount, bool deposit) external view returns (uint256);
.\interfaces\IBuoy.sol:    function stableToLp(uint256[3] calldata inAmount, bool deposit) external view returns (uint256);
.\interfaces\IController.sol:    function stablecoins() external view returns (address[3] memory);
.\interfaces\IController.sol:    function vaults() external view returns (address[3] memory);
.\interfaces\ICurve.sol:    function calc_token_amount(uint256[3] calldata inAmounts, bool deposit) external view returns (uint256);
.\interfaces\ICurve.sol:    function add_liquidity(uint256[3] calldata uamounts, uint256 min_mint_amount) external;
.\interfaces\ICurve.sol:    function remove_liquidity(uint256 amount, uint256[3] calldata min_uamounts) external;
.\interfaces\ICurve.sol:    function remove_liquidity_imbalance(uint256[3] calldata amounts, uint256 max_burn_amount) external;
.\interfaces\IDepositHandler.sol:        uint256[3] calldata inAmounts,
.\interfaces\IDepositHandler.sol:        uint256[3] calldata inAmounts,
.\interfaces\IExposure.sol:    function getUnifiedAssets(address[3] calldata vaults)
.\interfaces\IExposure.sol:        returns (uint256 unifiedTotalAssets, uint256[3] memory unifiedAssets);
.\interfaces\IExposure.sol:        uint256[3] calldata unifiedAssets,
.\interfaces\IExposure.sol:        uint256[3] calldata targetPercents
.\interfaces\IExposure.sol:    ) external pure returns (uint256[3] memory vaultIndexes);
.\interfaces\IExposure.sol:        uint256[3] calldata targets,
.\interfaces\IExposure.sol:        address[3] calldata vaults,
.\interfaces\IExposure.sol:    ) external view returns (uint256[3] memory);
.\interfaces\IInsurance.sol:    function calculateDepositDeltasOnAllVaults() external view returns (uint256[3] memory);
.\interfaces\IInsurance.sol:    function getDelta(uint256 withdrawUsd) external view returns (uint256[3] memory delta);
.\interfaces\IInsurance.sol:            uint256[3] memory,
.\interfaces\IInsurance.sol:            uint256[3] memory,
.\interfaces\IInsurance.sol:    function sortVaultsByDelta(bool bigFirst) external view returns (uint256[3] memory vaultIndexes);
.\interfaces\ILifeGuard.sol:    function getAssets() external view returns (uint256[3] memory);
.\interfaces\ILifeGuard.sol:    function distributeCurveVault(uint256 amount, uint256[3] memory delta) external returns (uint256[3] memory);
.\interfaces\ILifeGuard.sol:    function invest(uint256 whaleDepositAmount, uint256[3] calldata delta) external returns (uint256 dollarAmount);
.\interfaces\ILifeGuard.sol:        uint256[3] calldata inAmounts,
.\interfaces\IWithdrawHandler.sol:        uint256[3] calldata minAmounts
.\interfaces\IWithdrawHandler.sol:    function withdrawAllBalanced(bool pwrd, uint256[3] calldata minAmounts) external;
.\pools\oracle\Buoy3Pool.sol:    function getTokenRatios(uint256 i) private view returns (uint256[3] memory _ratios) {
.\pools\oracle\Buoy3Pool.sol:        uint256[3] memory _prices;
.\pools\oracle\Buoy3Pool.sol:        for (uint256 j = 0; j < 3; j++) {


## Tools Used

## Recommended Mitigation Steps
Do the following replacements
- 10**18 ==> DEFAULT_DECIMALS_FACTOR
- 5E17 ==> DEFAULT_DECIMALS_FACTOR /2 or BASE/2
- 10000 ==> PERCENTAGE_DECIMAL_FACTOR
- 10**4 ==> PERCENTAGE_DECIMAL_FACTOR
- 3 ==> N_COINS

