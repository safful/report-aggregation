## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Useless imports](https://github.com/code-423n4/2021-10-mochi-findings/issues/154) 

# Handle

pauliax


# Vulnerability details

## Impact
contract USDM does not need to import IERC3156FlashLender again as it was already imported in IUSDM.
  import "../interfaces/IERC3156FlashLender.sol";

contract DutchAuctionLiquidator makes no use of these imports:
  import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
  import "@uniswap/v2-periphery/contracts/interfaces/IUniswapV2Router02.sol";
  import "@mochifi/library/contracts/BeaconProxyDeployer.sol";

## Recommended Mitigation Steps
Consider reviewing all the unused imports and removing them to reduce the size of the contract and thus save some deployment gas.

