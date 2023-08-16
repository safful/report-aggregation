## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Uninitialized or Incorrectly set auctionInterval may lead to liquidation engine livelock](https://github.com/code-423n4/2021-05-yield-findings/issues/44) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The grab() function in Cauldron is used by the Witch or other liquidation engines to grab vaults that are under-collateralized. To prevent re-grabbing without sufficient time for auctioning collateral/debt, the logic uses an auctionInterval threshold to give a reasonable window to a liquidation engine that has grabbed the vault.

The grab() function has a comment on Line 354: “// Grabbing a vault protects it for a day from being grabbed by another liquidator. All grabbed vaults will be suddenly released on the 7th of February 2106, at 06:28:16 GMT. I can live with that.” indicating a requirement of the auctionInterval being equal to one day. This can happen only if the auctionInterval is set appropriately. However, this state variable is uninitialized (defaults to 0) and depends on setAuctionInterval() being called with the appropriate auctionInterval_ value which is also not validated.

Discussion with the project lead indicated that this comment is incorrect. Nevertheless, it is safer to initialize auctionInterval at declaration to a safe default value instead of the current 0 which will allow liquidation engines to re-grab vaults without making any progress on liquidation auction. It is also good to add a threshold check in setAuctionInterval() to ensure the new value meets/exceeds a reasonable default value.

Rationale for Medium-severity impact: While the likelihood of this may be low, the impact is high because liquidation engines will keep re-grabbing vaults from each other and potentially result in liquidation bots entering a live-lock situation without making any progress on liquidation auctions. This will result in collateral being stuck and impact entire protocol’s functioning. So, with low likelihood and high impact, the severity (according to OWASP) is medium.


## Proof of Concept

Configuration recipe forgets to set the auctionInterval state variable by calling setAuctionInterval() and inadvertently leaves it at the default value of 0. Alternatively, it calls it but with a lower than intended/reasonable auction interval value. Both scenarios fail to give sufficient protection to liquidation engines from having their grabbed vaults re-grabbed without sufficient time for liquidation auctions.

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L63

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L108-L115

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L354


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

1. Initialize auctionInterval  at declaration with a reasonable default value.
2. Add a threshold check in setAuctionInterval() to ensure the new value meets/exceeds a reasonable default value.

