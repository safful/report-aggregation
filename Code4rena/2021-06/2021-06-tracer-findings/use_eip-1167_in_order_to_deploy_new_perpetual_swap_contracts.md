## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use EIP-1167 in order to deploy new perpetual swap contracts](https://github.com/code-423n4/2021-06-tracer-findings/issues/120) 

# Handle

a_delamo


# Vulnerability details

## Impact

For every new TracerPerpetualSwaps contract, we need to deploy a new Liquidation, Insurance, and Pricing contract. 
All these deployments are really gas-intensive, so it would be recommended to use EIP-1167: Minimal Proxy Contract to reduce the gas cost of the deployments.

```solidity
function _deployTracer(
        bytes calldata _data,
        address tracerOwner,
        address oracle,
        address fastGasOracle,
        uint256 maxLiquidationSlippage
    ) internal returns (address) {
        // Create and link tracer to factory
        address market = IPerpsDeployer(perpsDeployer).deploy(_data);
        ITracerPerpetualSwaps tracer = ITracerPerpetualSwaps(market);

        validTracers[market] = true;
        tracersByIndex[tracerCounter] = market;
        tracerCounter++;

        // Instantiate Insurance contract for tracer
        address insurance = IInsuranceDeployer(insuranceDeployer).deploy(market);
        address pricing = IPricingDeployer(pricingDeployer).deploy(market, insurance, oracle);
        address liquidation = ILiquidationDeployer(liquidationDeployer).deploy(
            pricing,
            market,
            insurance,
            fastGasOracle,
            maxLiquidationSlippage
        );

        // Perform admin operations on the tracer to finalise linking
        tracer.setInsuranceContract(insurance);
        tracer.setPricingContract(pricing);
        tracer.setLiquidationContract(liquidation);

        // Ownership either to the deployer or the DAO
        tracer.transferOwnership(tracerOwner);
        ILiquidation(liquidation).transferOwnership(tracerOwner);
        emit TracerDeployed(tracer.marketId(), address(tracer));
        return market;
    }
```


More info:
https://blog.openzeppelin.com/deep-dive-into-the-minimal-proxy-contract/
https://eips.ethereum.org/EIPS/eip-1167


