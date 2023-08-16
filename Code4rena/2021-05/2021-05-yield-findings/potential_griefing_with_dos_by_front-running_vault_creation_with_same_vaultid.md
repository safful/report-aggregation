## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Potential griefing with DoS by front-running vault creation with same vaultID](https://github.com/code-423n4/2021-05-yield-findings/issues/43) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The vaultID for a new vault being built is required to be specified by the user building a vault via the build() function (instead of being assigned by the Cauldron/protocol). An attacker can observe a build() as part of a batch transaction in the mempool, identify the vaultID being requested and front-run that by constructing a malicious batch transaction with only the build operation with that same vaultID. The protocol would create a vault with that vaultID and assign attacker as its owner. More importantly, the valid batch transaction in the mempool which was front-run will later fail to create its vault because that vaultID already exists, as per the check on Line180 of Cauldron.sol. As a result, the valid batch transaction fails entirely because of the attacker front-running with the observed vaultID.

While the attacker gains nothing except the ownership of an empty vault after spending the gas, this could grief the protocol’s real users by preventing them from opening a vault and interacting with the protocol in any manner.

Rationale for Medium-severity impact: While the likelihood of this may be low, the impact is high because valid vaults from the Yield front-end will never be successfully created and will lead to a DoS against the entire protocol’s functioning. So, with low likelihood and high impact, the severity (according to OWASP) is medium.

## Proof of Concept

Alice uses Yield’s front-end to create a valid batch transaction. Evil Eve observes that in the mempool and identifies the vaultID of the vault being built by Alice. Eve submits her own batch transaction (without using the front-end) with only a build operation using Alice’s vaultID. She uses a higher gas price to front-run Alice’s transaction and get’s the protocol to assign that vaultID to herself. Alice’s batch transaction later fails because the vaultID she requested is already assigned to Eve. Eve can do this for any valid transaction to grief protocol users by wasting her gas to cause DoS.

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L180

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L173-L190

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L133-L135

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Ladle.sol#L249-L255


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Mitigate this DoS vector by having the Cauldron assign the vauldID instead of user specifying it in the build() operation. This would likely require the build() to be a separate non-batch transaction followed by other operations that use the vaultID assigned in build(). Consider the pros/cons of this approach because it will significantly affect the batching/caching logic in Ladle.

Alternatively, consider adding validation logic in Ladle’s batching to revert batches that have only build or a subset of the operations that do not make sense to the protocol’s operations per valid recipes, which could be an attacker’s signature pattern.

