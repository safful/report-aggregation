## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [redundant check of array length](https://github.com/code-423n4/2021-06-gro-findings/issues/12) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _stableToUsd and _stableToLp check that the size of the input array is right.
However because that parameter definition also contains the length (e.g. [N_COINS] ), it is already checked by solidity.

So checking it again is not necessary.
Note: if this would be necessary than it should also be done at the other functions that have an input parameter with  [N_COINS], see at Proof of concept.

## Proof of Concept
//https://github.com/code-423n4/2021-06-gro/blob/main/contracts/pools/oracle/Buoy3Pool.sol#L174
    function _stableToUsd(uint256[N_COINS] memory tokenAmounts, bool deposit) internal view returns (uint256) {
        require(tokenAmounts.length == N_COINS, "deposit: !length");
    ...

    function _stableToLp(uint256[N_COINS] memory tokenAmounts, bool deposit) internal view returns (uint256) {
        require(tokenAmounts.length == N_COINS, "deposit: !length");
      ..

Other functions with a [N_COINS] parameter:
.\Controller.sol:    function distributeCurveAssets(uint256 amount, uint256[N_COINS] memory delta) external onlyWhitelist {
.\DepositHandler.sol:    function _invest(uint256[N_COINS] memory _inAmounts, uint256 roughUsd) internal returns (uint256 dollarAmount) {
.\DepositHandler.sol:    function roughUsd(uint256[N_COINS] memory inAmounts) private view returns (uint256 usdAmount) {
.\WithdrawHandler.sol:    function withdrawAllBalanced(bool pwrd, uint256[N_COINS] calldata minAmounts) external override {
.\insurance\Exposure.sol:    function getUnifiedAssets(address[N_COINS] calldata vaults)
.\insurance\Exposure.sol:    function calculateStableCoinExposure(uint256[N_COINS] memory directlyExposure, uint256 curveExposure)
.\insurance\Insurance.sol:    function calculateWithdrawalAmountsOnPartVaults(uint256 amount, address[N_COINS] memory vaults)
.\insurance\Insurance.sol:    function calculateWithdrawalAmountsOnAllVaults(uint256 amount, address[N_COINS] memory vaults)
.\pools\LifeGuard3Pool.sol:    function distributeCurveVault(uint256 amount, uint256[N_COINS] memory delta)
.\pools\LifeGuard3Pool.sol:    function invest(uint256 depositAmount, uint256[N_COINS] calldata delta)
.\pools\LifeGuard3Pool.sol:    function _withdrawUnbalanced(uint256 inAmount, uint256[N_COINS] memory delta) private {
.\pools\oracle\Buoy3Pool.sol:    function stableToUsd(uint256[N_COINS] calldata inAmounts, bool deposit) external view override returns (uint256) {
.\pools\oracle\Buoy3Pool.sol:    function stableToLp(uint256[N_COINS] calldata tokenAmounts, bool deposit) external view override returns (uint256) {

## Tools Used

## Recommended Mitigation Steps
Remove :
require(tokenAmounts.length == N_COINS, "deposit: !length");

