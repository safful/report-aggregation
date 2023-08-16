## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [WhitelistPeriodManager: Improper state handling of exclusion removals](https://github.com/code-423n4/2022-03-biconomy-findings/issues/72) 

# Lines of code

https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/WhitelistPeriodManager.sol#L178-L184
https://github.com/code-423n4/2022-03-biconomy/blob/main/contracts/hyphen/WhitelistPeriodManager.sol#L115-L125


# Vulnerability details

## Impact

The `totalLiquidity` and `totalLiquidityByLp` mappings are not updated when an address is removed from the `isExcludedAddress` mapping. While this affects the enforcement of the cap limits and the `getMaxCommunityLpPositon()` function, the worst impact this has is that the address cannot have liquidity removed / transferred due to subtraction overflow.

In particular, users can be prevented from withdrawing their staked LP tokens from the liquidity farming contract should it become non-excluded.

## Proof of Concept

- Assume liquidity farming address `0xA` is excluded
- Bob stakes his LP token
- Liquidity farming contract is no longer to be excluded: `setIsExcludedAddressStatus([0xA, false])`
- Bob attempts to withdraw liquidity → reverts because `totalLiquidityByLp[USDC][0xA] = 0`, resulting in subtraction overflow.

```jsx
// insert test case in Withdraw test block of LiquidityFarming.tests.ts
it.only('will brick withdrawals by no longer excluding farming contract', async () => {
  await farmingContract.deposit(1, bob.address);
  await wlpm.setIsExcludedAddressStatus([farmingContract.address], [false]);
  await farmingContract.connect(bob).withdraw(1, bob.address);
});

// results in
// Error: VM Exception while processing transaction: reverted with panic code 0x11 (Arithmetic operation underflowed or overflowed outside of an unchecked block)
```

## Recommended Mitigation Steps

The simplest way is to prevent exclusion removals.

```jsx
function setIsExcludedAddresses(address[] memory _addresses) external onlyOwner {
  for (uint256 i = 0; i < _addresses.length; ++i) {
    isExcludedAddress[_addresses[i]] = true;
    // emit event
    emit AddressExcluded(_addresses[i]);
  }
}
```

