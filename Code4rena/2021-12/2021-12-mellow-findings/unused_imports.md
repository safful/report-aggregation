## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2021-12-mellow-findings/issues/1) 

# Handle

robee


# Vulnerability details

In the following files there are contract imports that aren't used. 
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore). 
The following is a full list of all unused imports, we went through the whole code to find it :) <solidity file> <line number> <actual import line>: 

        AaveVaultGovernance.sol, line 3, import "./interfaces/IProtocolGovernance.sol";
        ERC20Vault.sol, line 3, import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
        ERC20VaultGovernance.sol, line 4, import "./interfaces/IProtocolGovernance.sol";
        GatewayVaultGovernance.sol, line 3, import "./interfaces/IProtocolGovernance.sol";
        ILpIssuer.sol, line 3, import "./IVault.sol";
        IProtocolGovernance.sol, line 4, import "./IVaultRegistry.sol";
        IVaultFactory.sol, line 3, import "./IVaultGovernance.sol";
        IVaultGovernance.sol, line 3, import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
        IVaultRegistry.sol, line 5, import "./IVaultFactory.sol";
        IVaultRegistry.sol, line 6, import "./IVaultGovernance.sol";
        LpIssuer.sol, line 9, import "./interfaces/IProtocolGovernance.sol";
        LpIssuer.sol, line 11, import "./DefaultAccessControl.sol";
        LpIssuerGovernance.sol, line 4, import "./interfaces/IProtocolGovernance.sol";
        AaveVaultGovernanceTest.sol, line 3, import "../interfaces/IAaveVaultGovernance.sol";
        ERC20VaultTest.sol, line 4, import "../interfaces/IVaultFactory.sol";
        GatewayVaultTest.sol, line 4, import "../interfaces/IVaultFactory.sol";
        TestEncoding.sol, line 4, import "../interfaces/IVaultGovernance.sol";
        TestEncoding.sol, line 5, import "../interfaces/IVaultRegistry.sol";
        TestFunctionEncoding.sol, line 4, import "../interfaces/IVaultGovernance.sol";
        TestFunctionEncoding.sol, line 5, import "../interfaces/IVault.sol";
        UniV3VaultGovernanceTest.sol, line 3, import "../interfaces/IUniV3VaultGovernance.sol";
        UniV3VaultTest.sol, line 4, import "../interfaces/IVaultFactory.sol";
        IChiefTrader.sol, line 3, import "../../interfaces/IProtocolGovernance.sol";
        UniV3Vault.sol, line 3, import "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";
        UniV3Vault.sol, line 5, import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
        UniV3VaultGovernance.sol, line 3, import "./interfaces/IProtocolGovernance.sol";
        VaultGovernance.sol, line 3, import "@openzeppelin/contracts/token/ERC721/IERC721.sol";
        VaultRegistry.sol, line 6, import "./interfaces/IVaultFactory.sol";
        YearnVault.sol, line 3, import "./interfaces/external/aave/ILendingPool.sol";
        YearnVault.sol, line 7, import "./libraries/ExceptionsLibrary.sol";
        YearnVaultGovernance.sol, line 3, import "@openzeppelin/contracts/utils/structs/EnumerableSet.sol";
        YearnVaultGovernance.sol, line 4, import "./interfaces/IProtocolGovernance.sol";
        YearnVaultGovernance.sol, line 7, import "./libraries/ExceptionsLibrary.sol";


