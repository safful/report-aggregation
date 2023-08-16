## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Init function can be called by everyone](https://github.com/code-423n4/2021-04-vader-findings/issues/18) 

# Handle

gpersoon


# Vulnerability details

## Impact
Most of the solidity contracts have an init function that everyone can call.
This could lead to a race condition when the contract is deployed. At that moment a hacker could call the init function and make the deployed contracts useless. Then it would have to be redeployed, costing a lot of gas.

## Proof of Concept

DAO.sol:    function init(address _vader, address _usdv, address _vault) public {
Factory.sol:    function init(address _pool) public {
Pools.sol:    function init(address _vader, address _usdv, address _router, address _factory) public {
Router.sol:    function init(address _vader, address _usdv, address _pool) public {
USDV.sol:    function init(address _vader, address _vault, address _router) external {
Utils.sol:    function init(address _vader, address _usdv, address _router, address _pools, address _factory) public {
Vader.sol:    function init(address _vether, address _USDV, address _utils) external {
Vault.sol:    function init(address _vader, address _usdv, address _router, address _factory, address _pool) public {
 

## Tools Used
Editor

## Recommended Mitigation Steps
Add a check to the init function, for example that only the deployer can call the function.

