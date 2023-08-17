## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- satisfactory
- selected for report
- M-03

# [Changing default reserved token beneficiary may result in wrong beneficiary for tier](https://github.com/code-423n4/2022-10-juicebox-findings/issues/63) 

# Lines of code

https://github.com/jbx-protocol/juice-nft-rewards/blob/89cea0e2a942a9dc9e8d98ae2c5f1b8f4d916438/contracts/JBTiered721DelegateStore.sol#L701


# Vulnerability details

## Impact
When the `reservedTokenBeneficiary` of a tier is equal to `defaultReservedTokenBeneficiaryOf[msg.sender]`, it is not explicitly set for this tier. This generally works well because in the function `reservedTokenBeneficiaryOf(address _nft, uint256 _tierId)`, `defaultReservedTokenBeneficiaryOf[_nft]` is used as a backup when `_reservedTokenBeneficiaryOf[_nft][_tierId]` is not set. However, it will lead to the wrong beneficiary when `defaultReservedTokenBeneficiaryOf[msg.sender]` is later changed, as this new beneficiary will be used for the tier, which is not the intended one.

## Proof Of Concept
`defaultReservedTokenBeneficiaryOf[address(delegate)]` is originally set to `address(Bob)` when the following happens:
1.) A new tier 42 is added with `_tierToAdd.reservedTokenBeneficiary = address(Bob)`. Because this is equal to `defaultReservedTokenBeneficiaryOf[address(delegate)]`, `_reservedTokenBeneficiaryOf[msg.sender][_tierId]` is not set.
2.) The owner calls `setDefaultReservedTokenBeneficiary` to change the default beneficiary (i.e., the value `defaultReservedTokenBeneficiaryOf[address(delegate)]`) to `address(Alice)`.
3.) Now, every call to `reservedTokenBeneficiaryOf(address(delegate), 42)` will return `address(Alice)`, meaning she will get these reserved tokens. This is of course wrong, the tier was explicitly created with Bob as the beneficiary.

## Recommended Mitigation Steps
Also set `_reservedTokenBeneficiaryOf[msg.sender][_tierId]` when it is equal to the current default beneficiary.