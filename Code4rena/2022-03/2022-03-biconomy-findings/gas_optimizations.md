## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-03-biconomy-findings/issues/11) 

Title: Unnecessary Reentrancy Guards
Severity: GAS

Where there is onlyOwner or Initializer modifer, the reentrancy gaurd isn't 
necessary (unless you don't trust the owner or the deployer, which will lead to full security breakdown of the project and we believe this is not the case)
This is a list we found of such occurrences: 

        LiquidityFarming.sol no need both nonReentrant and onlyOwner modifiers in reclaimTokens



Title: Change transferFrom to transfer
Severity: GAS

'transferFrom(address(this), *, **)' could be replaced by the following more gas efficient 'transfer(*, **)'
               This replacement is more gas efficient and improves the code quality.

        LiquidityFarming.sol, 250 : lpToken.safeTransferFrom(address(this), msgSender, _nftId);


Title: Public functions to external
Severity: GAS

The following functions could be set external to save gas and improve code quality. 
External call cost is less expensive than of public functions. 

        ExecutorManager.sol, getAllExecutors
        LPToken.sol, initialize
        LPToken.sol, exists
        WhitelistPeriodManager.sol, initialize


Title: Use unchecked to save gas for certain additive calculations that cannot overflow
Severity: GAS


You can use unchecked in the following calculations since there is no risk to overflow:

        LiquidityFarming.sol (L#275) - accumulator += rewardRateLog[_baseToken][i].rewardsPerSecond * (counter - max(lastUpdatedTime, rewardRateLog[_baseToken][i].timestamp));



Title: State variables that could be set immutable
Severity: GAS

In the following files there are state variables that could be set immutable to save gas. 

        liquidityProviders in LiquidityFarming.sol
        tokenManager in LiquidityPool.sol
        lpToken in LiquidityFarming.sol



Title: Use calldata instead of memory
Severity: GAS


Use calldata instead of memory for function parameters
In some cases, having function arguments in calldata instead of
memory is more optimal.
    

        LiquidityPool.depositErc20 (tag)
        LPToken.initialize (_symbol)
        LiquidityPool.sendFundsToUser (depositHash)
        LiquidityPool.permitEIP2612AndDepositErc20 (tag)
        LiquidityPool.permitAndDepositErc20 (tag)
        LiquidityPool.depositNative (tag)
        LPToken.updateTokenMetadata (_lpTokenMetadata)
        LiquidityPool.checkHashStatus (depositHash)
        LPToken.initialize (_name)



Title: Caching array length can save gas
Severity: GAS


Caching the array length is more gas efficient.
This is because access to a local variable in solidity is more efficient than query storage / calldata / memory.
We recommend to change from:    

    for (uint256 i=0; i<array.length; i++) { ... }

to: 

    uint len = array.length  
    for (uint256 i=0; i<len; i++) { ... }


        ExecutorManager.sol, executorArray, 31
        TokenManager.sol, tokenConfig, 78
        ExecutorManager.sol, executorArray, 47
        WhitelistPeriodManager.sol, _tokens, 228
        LPToken.sol, nftIds, 77
        WhitelistPeriodManager.sol, _addresses, 180



Title: Storage double reading. Could save SLOAD
Severity: GAS

Reading a storage variable is gas costly (SLOAD). In cases of multiple read of a storage variable in the same scope, caching the first read (i.e saving as a local variable) can save gas and decrease the
 overall gas uses. The following is a list of functions and the storage variables that you read twice: 

        LiquidityProviders.sol: BASE_DIVISOR is read twice in _increaseLiquidity



Title: Unnecessary index init
Severity: GAS


In for loops you initialize the index to start from 0, but it already initialized to 0 in default and this assignment cost gas. 
It is more clear and gas efficient to declare without assigning 0 and will have the same meaning:

        WhitelistPeriodManager.sol, 180
        WhitelistPeriodManager.sol, 228
        LPToken.sol, 77
        ExecutorManager.sol, 31
        ExecutorManager.sol, 47
        TokenManager.sol, 78




Title: Consider inline the following functions to save gas
Severity: GAS


    You can inline the following functions instead of writing a specific function to save gas.
    (see https://github.com/code-423n4/2021-11-nested-findings/issues/167 for a similar issue.)

    
        TokenManager.sol, _msgData, { return ERC2771Context._msgData(); }
        TokenManager.sol, _msgSender, { return ERC2771Context._msgSender(); }
        WhitelistPeriodManager.sol, _msgData, { return ERC2771ContextUpgradeable._msgData(); }
        LiquidityPool.sol, _msgData, { return ERC2771ContextUpgradeable._msgData(); }
        LiquidityFarming.sol, _msgData, { return ERC2771ContextUpgradeable._msgData(); }
        LiquidityProviders.sol, _msgData, { return ERC2771ContextUpgradeable._msgData(); }
        LPToken.sol, _msgData, { return ERC2771ContextUpgradeable._msgData(); }
        LiquidityPool.sol, _msgSender, { return ERC2771ContextUpgradeable._msgSender(); }



Title: Use bytes32 instead of string to save gas whenever possible
Severity: GAS


    Use bytes32 instead of string to save gas whenever possible.
    String is a dynamic data structure and therefore is more gas consuming then bytes32.

    
        LPToken.sol (L140), string memory json = Base64.encode( bytes( string( abi.encodePacked( '{"name": "', name(), '", "description": "', description, '", "image": "data:image/svg+xml;base64,', Base64.encode(bytes(svgData)), '", "attributes": ', attributes, "}" ) ) ) );




Title: Unused state variables
Severity: GAS

Unused state variables are gas consuming at deployment (since they are located in storage) and are 
a bad code practice. Removing those variables will decrease deployment gas cost and improve code quality. 
This is a full list of all the unused storage variables we found in your code base. 

        LPToken.sol, NATIVE



Title: Prefix increments are cheaper than postfix increments
Severity: GAS

Prefix increments are cheaper than postfix increments. 
Further more, using unchecked {++x} is even more gas efficient, and the gas saving accumulates every iteration and can make a real change
There is no risk of overflow caused by increamenting the iteration index in for loops (the `++i` in `for (uint256 i = 0; i < numIterations; ++i)`).
But increments perform overflow checks that are not necessary in this case.
These functions use not using prefix increments (`++x`) or not using the unchecked keyword: 

        just change to unchecked: ExecutorManager.sol, i, 47
        just change to unchecked: LiquidityFarming.sol, index, 233
        just change to unchecked: LPToken.sol, i, 77
        just change to unchecked: WhitelistPeriodManager.sol, i, 228
        just change to unchecked: WhitelistPeriodManager.sol, i, 248
        just change to unchecked: TokenManager.sol, index, 78
        just change to unchecked: WhitelistPeriodManager.sol, i, 180
        just change to unchecked: ExecutorManager.sol, i, 31



Title: Unused inheritance
Severity: GAS


    Some of your contract inherent contracts but aren't use them at all.
    We recommend not to inherent those contracts.
    
        TokenManager.sol; the inherited contracts Pausable not used


Title: Internal functions to private
Severity: GAS

The following functions could be set private to save gas and improve code quality:

        WhitelistPeriodManager.sol, _msgData
        LiquidityProviders.sol, _addLiquidity
        WhitelistPeriodManager.sol, _isSupportedToken


Title: Unused inheritance
Severity: GAS


    Some of your contract inherent contracts but aren't use them at all.
    We recommend not to inherent those contracts.
    
        TokenManager.sol; the inherited contracts Pausable not used


Title: Unused imports
Severity: GAS


In the following files there are contract imports that aren't used
Import of unnecessary files costs deployment gas (and is a bad coding practice that is important to ignore)

        LiquidityPool.sol, line 5, import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
        WhitelistPeriodManager.sol, line 6, import "@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
        WhitelistPeriodManager.sol, line 5, import "@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";




Title: Use != 0 instead of > 0
Severity: GAS


Using != 0 is slightly cheaper than > 0. (see https://github.com/code-423n4/2021-12-maple-findings/issues/75 for similar issue)


        LiquidityProviders.sol, 283: change '_amount > 0' to '_amount != 0'
        LiquidityPool.sol, 401: change 'balance > 0' to 'balance != 0'
        LiquidityPool.sol, 292: change 'balance > 0' to 'balance != 0'