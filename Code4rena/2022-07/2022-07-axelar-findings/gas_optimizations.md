## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-07-axelar-findings/issues/18) 

- In `refundTokenDeposit` within `AxelarDepositService`, the address of the `gatewayToken` is retrieved in every loop iteration (https://github.com/code-423n4/2022-07-axelar/blob/a46fa61e73dd0f3469c0263bc6818e682d62fb5f/contracts/deposit-service/AxelarDepositService.sol#L115). However, as this address does not change, the retrieval can be moved outside of the loop to save a lot of gas when the number of tokens is large.
- In multiple for loops, the loop iteration can be marked as `unchecked` because an overflow is not possible (as the iterator is bounded):
```
./auth/AxelarAuthWeighted.sol:        for (uint256 i; i < recentOperators.length; ++i) {
./auth/AxelarAuthWeighted.sol:        for (uint256 i = 0; i < weightsLength; ++i) {
./auth/AxelarAuthWeighted.sol:        for (uint256 i = 0; i < signatures.length; ++i) {
./auth/AxelarAuthWeighted.sol:            for (; operatorIndex < operatorsLength && signer != operators[operatorIndex]; ++operatorIndex) {}
./auth/AxelarAuthWeighted.sol:        for (uint256 i; i < accounts.length - 1; ++i) {
./gas-service/AxelarGasService.sol:        for (uint256 i; i < tokens.length; i++) {
./deposit-service/AxelarDepositService.sol:        for (uint256 i; i < refundTokens.length; i++) {
./deposit-service/AxelarDepositService.sol:        for (uint256 i; i < refundTokens.length; i++) {
./deposit-service/AxelarDepositService.sol:        for (uint256 i; i < refundTokens.length; i++) {
```