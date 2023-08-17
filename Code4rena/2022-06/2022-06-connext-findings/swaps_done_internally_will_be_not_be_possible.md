## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Swaps done internally will be not be possible](https://github.com/code-423n4/2022-06-connext-findings/issues/249) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/BridgeFacet.sol#L346
https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/BridgeFacet.sol#L812


# Vulnerability details



Affected  functions(that rely on swapAsset()) are:

[https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/AssetLogic.sol#L193](https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/AssetLogic.sol#L193)

[https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/AssetLogic.sol#L159](https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/AssetLogic.sol#L159)

swapAsset() facilitates two swaps, either using the internal or external pool. But if an internal pool exists, a swap will be unsuccessful because the call to

s.swapStorages[_canonicalId].swapInternal() takes two incorrect arguments (due to an incorrect ordering, this seemed to be an oversight, acknowledged by #Layne) :

[https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/AssetLogic.sol#L278-L279](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/AssetLogic.sol#L278-L279)

Based on the above mentioned code , the arguments would be incorrectly changed to :

[https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/SwapUtils.sol#L744-L745](https://github.com/code-423n4/2022-06-connext/blob/main/contracts/contracts/core/connext/libraries/SwapUtils.sol#L744-L745)

The condition checked here:

[https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/SwapUtils.sol#L750](https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/libraries/SwapUtils.sol#L750)

will never be true as the msg.sender would never own the quantity of tokens being swapped from since it's the wrong token.

