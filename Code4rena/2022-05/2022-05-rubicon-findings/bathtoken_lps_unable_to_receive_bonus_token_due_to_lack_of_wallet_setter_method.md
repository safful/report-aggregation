## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [BathToken LPs Unable To Receive Bonus Token Due To Lack Of Wallet Setter Method](https://github.com/code-423n4/2022-05-rubicon-findings/issues/107) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L629


# Vulnerability details

## Background

BathBuddy is a Vesting Wallet that payout withdrawers any `bonusTokens` they may have accrued while staking in the Bath Token (e.g. network incentives/governance tokens).

BathBuddy Vesting Wallet releases a user their relative share of the pool’s total vested bonus token during the withdraw call on BathToken.sol. This vesting occurs linearly over Unix time.

It was observed that the BathToken LPs are unable to receive any bonus tokens from the BathBuddy Vesting Wallet during withdraw and the bonus tokens are struck in the BathBuddy Vesting Wallet.

## Proof-of-Concept

The following shows that the address of the BathBuddy Vesting Wallet is stored in the `rewardsVestingWallet` state variable and it is used to call the `release` function to distribute bonus to the BathToken withdrawers.

[https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L629](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L629)

```solidity
function distributeBonusTokenRewards(
    address receiver,
    uint256 sharesWithdrawn,
    uint256 initialTotalSupply
) internal {
    if (bonusTokens.length > 0) {
        for (uint256 index = 0; index < bonusTokens.length; index++) {
            IERC20 token = IERC20(bonusTokens[index]);
            // Note: Shares already burned in Bath Token _withdraw

            // Pair each bonus token with a lightly adapted OZ Vesting wallet. Each time a user withdraws, they
            //  are released their relative share of this pool, of vested BathBuddy rewards
            // The BathBuddy pool should accrue ERC-20 rewards just like OZ VestingWallet and simply just release the withdrawer's relative share of releaseable() tokens
            if (rewardsVestingWallet != IBathBuddy(0)) {
                rewardsVestingWallet.release(
                    (token),
                    receiver,
                    sharesWithdrawn,
                    initialTotalSupply,
                    feeBPS
                );
            }
        }
    }
}
```

However, there is no setter method to initialise the value of the `rewardsVestingWallet` state variable in the contracts. Therefore, the value of `rewardsVestingWallet` will always be zero. Note that Solidity only create a default getter for public state variable, but does not create a default setter.

Since `rewardsVestingWallet` is always zero, the condition `if (rewardsVestingWallet != IBathBuddy(0))` will always be evaluated as `false`. Thus, the code block `rewardsVestingWallet.release` will never be reached.

## Impact

Loss of Fund for the users. BathToken LPs are not able to receive their `bonusToken`.

## Recommended Mitigation Steps

Implement a setter method for the `rewardsVestingWallet` state variable in the contracts so that it can be initialised with BathBuddy Vesting Wallet address.

