## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Code Style: consistency](https://github.com/code-423n4/2021-10-slingshot-findings/issues/67) 

# Handle

WatchPug


# Vulnerability details

1. Use of `uint256` vs `uint`;

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/Slingshot.sol#L65-L86

    ```solidity=65
    function executeTrades(
        address fromToken,
        address toToken,
        uint256 fromAmount,
        TradeFormat[] calldata trades,
        uint256 finalAmountMin,
        address depricated
    ) external nonReentrant payable {
        depricated;
        require(finalAmountMin > 0, "Slingshot: finalAmountMin cannot be zero");
        require(trades.length > 0, "Slingshot: trades cannot be empty");
        for(uint256 i = 0; i < trades.length; i++) {
            // Checks to make sure that module exists and is correct
            require(moduleRegistry.isModule(trades[i].moduleAddress), "Slingshot: not a module");
        }

        uint256 initialBalance = _getTokenBalance(toToken);
        _transferFromOrWrap(fromToken, _msgSender(), fromAmount);

        executioner.executeTrades(trades);

        uint finalBalance;
    ```

2. Error message prefixed by the module name.

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/ApprovalHandler.sol#L14-L17

    ```solidity=14
    modifier onlySlingshot() {
        require(isSlingshot(_msgSender()), "Adminable: not a SLINGSHOT_CONTRACT_ROLE");
        _;
    }
    ```

    `ApprovalHandler.sol#onlySlingshot()` The prefix `Adminable` in the error message should be `ApprovalHandler`.

    https://github.com/code-423n4/2021-10-slingshot/blob/9c0432cca2e43731d5a0ae9c151dacf7835b8719/contracts/ModuleRegistry.sol#L36-L38

    ```solidity=36
    function registerSwapModule(address _moduleAddress) public onlyAdmin {
        require(!modulesIndex[_moduleAddress], "oops module already exists");
        require(ISlingshotModule(_moduleAddress).slingshotPing(), "not a module");
    ```

    `ModuleRegistry.sol#registerSwapModule()` Error messages are not prefixed.

