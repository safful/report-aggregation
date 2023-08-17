## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Unable to redeem from Notional](https://github.com/code-423n4/2022-06-illuminate-findings/issues/341) 

# Lines of code

[Redeemer.sol#L193](https://github.com/code-423n4/2022-06-illuminate/blob/main/redeemer/Redeemer.sol#L193)


# Vulnerability details

## Impact

The ```maxRedeem``` function is a view function which only returns the balance of the ```Redeemer.sol``` contract. After this value is obtained, the PT is not redeemed from Notional. The user will be unable to redeem PT from Notional through ```Redeemer.sol```.

## Proof of Concept

Notional code:
```
    function maxRedeem(address owner) public view override returns (uint256) {
        return balanceOf(owner);
    }
```

## Recommmended Mitigation Steps

Call [```redeem```](https://github.com/notional-finance/wrapped-fcash/blob/019cfa20369d5e0d9e7a38fea936cc649704780d/contracts/wfCashERC4626.sol#L205) from Notional using the ```amount``` from ```maxRedeem``` as the ```shares``` input after the call to ```maxRedeem```.

