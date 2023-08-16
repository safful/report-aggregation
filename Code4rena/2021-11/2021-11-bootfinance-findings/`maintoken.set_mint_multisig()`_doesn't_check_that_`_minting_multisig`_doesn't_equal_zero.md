## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`MainToken.set_mint_multisig()` doesn't check that `_minting_multisig` doesn't equal zero](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/66) 

# Handle

pants


# Vulnerability details

The function `MainToken.set_mint_multisig()` doesn't check that `_minting_multisig` doesn't equal zero before it sets it as the new `minting_multisig`.

## Impact
This function can be invoked by mistake with the zero address as `_minting_multisig`, causing the system to lose its `minting_multisig` forever, without the option to set a new `minting_multisig`.

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Check that `_minting_multisig` doesn't equal zero before setting it as the new `minting_multisig`.

