## Tags

- bug
- 1 (Low Risk)
- sponsor acknowledged
- sponsor confirmed

# [Harvesting and Funding Is Not Checked When the Contract Is Halted](https://github.com/code-423n4/2021-09-yaxis-findings/issues/10) 

# Handle

defsec


# Vulnerability details

## Impact

During the manual code review, It has been observed that harvesting and fundings progress is not checked when the contract is halted. This can cause misfunctionality and locking user funds during the halt progress.

## Proof of Concept

1-) Navigate to "https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/controllers/Controller.sol" contract.
2-) Observe the following code on the Controller.sol.

Functions earn and HarvestStrategy
```
    function harvestStrategy(address _strategy,uint256 _estimatedWETH,uint256 _estimatedYAXIS)
        external
        override
        onlyHarvester
        onlyStrategy(_strategy)
       

    function earn(
        address _strategy,
        address _token,
        uint256 _amount
    )
        external
        override
        onlyStrategy(_strategy)
        onlyVault(_token)

```

## Tools Used

None 

## Recommended Mitigation Steps

Implement the notHalt modifier into the functions. Only withdraw functions should be allowed on the contract. 

