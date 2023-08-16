## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Mix of external and public function visibility with the same access modifier](https://github.com/code-423n4/2021-11-nested-findings/issues/29) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact

Functions in the same contract with the same access modifier (e.g. onlyOwner or onlyFactory) have have a mix of public and external visibility. 
Set their visibility to external to save gas.

Affected contracts (see ):
- NestedRecords
- NestedFactory
- FeeSplitter
- NestedAsset


## Tools Used
Visial Studio Code + Solidity Visual Developer (Plugin)

## Recommended Mitigation Steps 
Set the visibility to external to save gas.


Extract from Solidity Visual Developer (Plugin) of the Contracts and visibility:

|  Contract  |         Type        |       Bases      |                  |                 |
|:----------:|:-------------------:|:----------------:|:----------------:|:---------------:|
|     └      |  **Function Name**  |  **Visibility**  |  **Mutability**  |  **Modifiers**  |
||||||
| **NestedRecords** | Implementation | Ownable |||
| └ | <Constructor> | Public ❗️ | 🛑  |NO❗️ |
| └ | createRecord | Public ❗️ | 🛑  | onlyFactory |
| └ | updateHoldingAmount | Public ❗️ | 🛑  | onlyFactory |
| └ | getAssetTokens | Public ❗️ |   |NO❗️ |
| └ | freeHolding | Public ❗️ | 🛑  | onlyFactory |
| └ | store | External ❗️ | 🛑  | onlyFactory |
| └ | getAssetHolding | External ❗️ |   |NO❗️ |
| └ | setFactory | External ❗️ | 🛑  | onlyOwner |
| └ | updateLockTimestamp | External ❗️ | 🛑  | onlyFactory |
| └ | setMaxHoldingsCount | External ❗️ | 🛑  | onlyOwner |
| └ | getAssetReserve | External ❗️ |   |NO❗️ |
| └ | getAssetTokensLength | External ❗️ |   |NO❗️ |
| └ | getLockTimestamp | External ❗️ |   |NO❗️ |
| └ | setReserve | External ❗️ | 🛑  | onlyFactory |
| └ | removeNFT | External ❗️ | 🛑  | onlyFactory |
| └ | deleteAsset | Public ❗️ | 🛑  | onlyFactory |
| └ | freeToken | Private 🔐 | 🛑  | |
||||||
| **NestedFactory** | Implementation | INestedFactory, ReentrancyGuard, Ownable, MixinOperatorResolver, Multicall |||
| └ | <Constructor> | Public ❗️ | 🛑  | MixinOperatorResolver |
| └ | <Receive Ether> | External ❗️ |  💵 |NO❗️ |
| └ | resolverAddressesRequired | Public ❗️ |   |NO❗️ |
| └ | addOperator | External ❗️ | 🛑  | onlyOwner |
| └ | removeOperator | External ❗️ | 🛑  | onlyOwner |
| └ | setReserve | External ❗️ | 🛑  | onlyOwner |
| └ | setFeeSplitter | External ❗️ | 🛑  | onlyOwner |
| └ | create | External ❗️ |  💵 | nonReentrant |
| └ | addTokens | External ❗️ |  💵 | nonReentrant onlyTokenOwner |
| └ | swapTokenForTokens | External ❗️ | 🛑  | nonReentrant onlyTokenOwner isUnlocked |
| └ | sellTokensToNft | External ❗️ | 🛑  | nonReentrant onlyTokenOwner isUnlocked |
| └ | sellTokensToWallet | External ❗️ | 🛑  | nonReentrant onlyTokenOwner isUnlocked |
| └ | destroy | External ❗️ | 🛑  | nonReentrant onlyTokenOwner isUnlocked |
| └ | withdraw | External ❗️ | 🛑  | nonReentrant onlyTokenOwner isUnlocked |
| └ | increaseLockTimestamp | External ❗️ | 🛑  | onlyTokenOwner |
| └ | unlockTokens | External ❗️ | 🛑  | onlyOwner |
| └ | _submitInOrders | Private 🔐 | 🛑  | |
| └ | _submitOutOrders | Private 🔐 | 🛑  | |
| └ | _submitOrder | Private 🔐 | 🛑  | |
| └ | _safeSubmitOrder | Private 🔐 | 🛑  | |
| └ | _transferToReserveAndStore | Private 🔐 | 🛑  | |
| └ | _transferInputTokens | Private 🔐 | 🛑  | |
| └ | _handleUnderSpending | Private 🔐 | 🛑  | |
| └ | _transferFeeWithRoyalty | Private 🔐 | 🛑  | |
| └ | _decreaseHoldingAmount | Private 🔐 | 🛑  | |
| └ | _safeTransferAndUnwrap | Private 🔐 | 🛑  | |
| └ | _safeTransferWithFees | Private 🔐 | 🛑  | |
| └ | _calculateFees | Private 🔐 |   | |
||||||
| **FeeSplitter** | Implementation | Ownable, ReentrancyGuard |||
| └ | <Constructor> | Public ❗️ | 🛑  |NO❗️ |
| └ | <Receive Ether> | External ❗️ |  💵 |NO❗️ |
| └ | getAmountDue | Public ❗️ |   |NO❗️ |
| └ | setRoyaltiesWeight | Public ❗️ | 🛑  | onlyOwner |
| └ | setShareholders | Public ❗️ | 🛑  | onlyOwner |
| └ | releaseToken | Public ❗️ | 🛑  | nonReentrant |
| └ | releaseTokens | External ❗️ | 🛑  |NO❗️ |
| └ | releaseETH | External ❗️ | 🛑  | nonReentrant |
| └ | sendFees | External ❗️ | 🛑  | nonReentrant |
| └ | sendFeesWithRoyalties | External ❗️ | 🛑  | nonReentrant |
| └ | updateShareholder | External ❗️ | 🛑  | onlyOwner |
| └ | totalShares | External ❗️ |   |NO❗️ |
| └ | totalReleased | External ❗️ |   |NO❗️ |
| └ | shares | External ❗️ |   |NO❗️ |
| └ | released | External ❗️ |   |NO❗️ |
| └ | findShareholder | External ❗️ |   |NO❗️ |
| └ | _sendFees | Private 🔐 | 🛑  | |
| └ | _addShares | Private 🔐 | 🛑  | |
| └ | _releaseToken | Private 🔐 | 🛑  | |
| └ | _addShareholder | Private 🔐 | 🛑  | |
| └ | _computeShareCount | Private 🔐 |   | |
||||||
| **NestedAsset** | Implementation | ERC721Enumerable, Ownable |||
| └ | <Constructor> | Public ❗️ | 🛑  | ERC721 |
| └ | tokenURI | Public ❗️ |   |NO❗️ |
| └ | originalOwner | Public ❗️ |   |NO❗️ |
| └ | mint | Public ❗️ | 🛑  | onlyFactory |
| └ | mintWithMetadata | External ❗️ | 🛑  | onlyFactory |
| └ | backfillTokenURI | External ❗️ | 🛑  | onlyFactory onlyTokenOwner |
| └ | burn | External ❗️ | 🛑  | onlyFactory onlyTokenOwner |
| └ | setFactory | External ❗️ | 🛑  | onlyOwner |
| └ | removeFactory | External ❗️ | 🛑  | onlyOwner |
| └ | _setTokenURI | Internal 🔒 | 🛑  | |

 Legend

|  Symbol  |  Meaning  |
|:--------:|-----------|
|    🛑    | Function can modify state |
|    💵    | Function is payable |


