## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Single Error Within SponsorVault Contract Could Cause Entire Cross-Chain Communication To Break Down](https://github.com/code-423n4/2022-06-connext-findings/issues/146) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L819](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L819


# Vulnerability details

## Proof-of-Concept

A third party sponsor would need to implement a `SponsorVault` contract that is aligned with the `ISponsorVault` interface.

Assume that a `SponsorVault` contract has been defined on Optimism chain. All cross-chain communications are required to call the `BridgeFacet.execute`, which in turn will trigger the `BridgeFacet._handleExecuteTransaction` internal function. 

However, if there is an error within `SponsorVault` contract in Optimism causing a revert when `s.sponsorVault.reimburseLiquidityFees` or `s.sponsorVault.reimburseRelayerFees` is called, the entire `execute` transaction will revert. Since `execute` transaction always revert, any cross-chain communication between Optimism and other domains will fail.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L819](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L819)

```solidity
  /**
   * @notice Process the transfer, and calldata if needed, when calling `execute`
   * @dev Need this to prevent stack too deep
   */
  function _handleExecuteTransaction(
    ExecuteArgs calldata _args,
    uint256 _amount,
    address _asset, // adopted (or local if specified)
    bytes32 _transferId,
    bool _reconciled
  ) private returns (uint256) {
    // If the domain if sponsored
    if (address(s.sponsorVault) != address(0)) {
      // fast liquidity path
      if (!_reconciled) {
        // Vault will return the amount of the fee they sponsored in the native fee
        // NOTE: some considerations here around fee on transfer tokens and ensuring
        // there are no malicious `Vaults` that do not transfer the correct amount. Should likely do a
        // balance read about it

        uint256 starting = IERC20(_asset).balanceOf(address(this));
        uint256 sponsored = s.sponsorVault.reimburseLiquidityFees(_asset, _args.amount, _args.params.to);

        // Validate correct amounts are transferred
        if (IERC20(_asset).balanceOf(address(this)) != starting + sponsored) {
          revert BridgeFacet__handleExecuteTransaction_invalidSponsoredAmount();
        }

        _amount = _amount + sponsored;
      }

      // Should dust the recipient with the lesser of a vault-defined cap or the converted relayer fee
      // If there is no conversion available (i.e. no oracles for origin domain asset <> dest asset pair),
      // then the vault should just pay out the configured constant
      s.sponsorVault.reimburseRelayerFees(_args.params.originDomain, payable(_args.params.to), _args.params.relayerFee);
    }
    ..SNIP..
```

## Impact

It will result in denial of service. The `SponsorVault` contract, which belongs to a third-party, is a single point of failure for a domain.

## Recommended Mitigation Steps

This is a problem commonly encountered whenever a method of a smart contract calls another contract – we cannot rely on the other contract to work 100% of the time, and it is dangerous to assume that the external call will always be successful. Additionally, external smart contract might be vulnerable and compromised by an attacker. Even if the team has audited or review the SponsorVault before whitelisting them, some risk might still exist.

Therefore, it is recommended to implement a fail-safe design where failure of an external call to SponsorVault will not disrupt the cross-chain communication. Consider implementing a try-catch block as shown below. If there is any issue with the external `SponsorVault ` contract, no funds are reimbursed to the users in the worst case scenario, but the issue will not cause any impact to the cross-chain communication.

```diff
function _handleExecuteTransaction(
	ExecuteArgs calldata _args,
	uint256 _amount,
	address _asset, // adopted (or local if specified)
	bytes32 _transferId,
	bool _reconciled
) private returns (uint256) {
	// If the domain if sponsored
	if (address(s.sponsorVault) != address(0)) {
	  // fast liquidity path
	  if (!_reconciled) {
		// Vault will return the amount of the fee they sponsored in the native fee
		// NOTE: some considerations here around fee on transfer tokens and ensuring
		// there are no malicious `Vaults` that do not transfer the correct amount. Should likely do a
		// balance read about it

		uint256 starting = IERC20(_asset).balanceOf(address(this));
+		try s.sponsorVault.reimburseLiquidityFees(_asset, _args.amount, _args.params.to) returns (uint256 sponsored) {
+			// Validate correct amounts are transferred
+			if (IERC20(_asset).balanceOf(address(this)) != starting + sponsored) {
+			  revert BridgeFacet__handleExecuteTransaction_invalidSponsoredAmount();
+			}
+
+			_amount = _amount + sponsored;
+		} catch {}
	  }

	  // Should dust the recipient with the lesser of a vault-defined cap or the converted relayer fee
	  // If there is no conversion available (i.e. no oracles for origin domain asset <> dest asset pair),
	  // then the vault should just pay out the configured constant
+	  try s.sponsorVault.reimburseRelayerFees(_args.params.originDomain, payable(_args.params.to), _args.params.relayerFee) {} catch {}
	..SNIP..
```

