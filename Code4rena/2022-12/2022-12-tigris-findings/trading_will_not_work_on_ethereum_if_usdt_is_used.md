## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-07

# [Trading will not work on ethereum if USDT is used](https://github.com/code-423n4/2022-12-tigris-findings/issues/198) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L652


# Vulnerability details

## Impact

Traders will not be able to:
1. Initiate a market order
2. Add margin
3. Add to position
4. initiate limit order

If USDT is set as the margin asset and protocol is deployed on ethereum.

(Note: this issue was submitted after consulting with the sponsor even though currently there are no plans to deploy the platform on ethereum)

## Proof of Concept

`USDT` has a race condition protection mechanism on ethereum chain:
It does not allow users to change the allowance without first changing the allowance to 0. 

`approve` function in `USDT` on ethereum:
https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code#L205
```
    function approve(address _spender, uint _value) public onlyPayloadSize(2 * 32) {

        // To change the approve amount you first have to reduce the addresses`
        //  allowance to zero by calling `approve(_spender, 0)` if it is not
        //  already 0 to mitigate the race condition described here:
        //  https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
        require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)));

        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);
    }
```

in `Trading` if users use `USDT` as margin to:
1. Initiate a market order
2. Add margin
3. Add to position
4. initiate limit order

The transaction will revert. 

This is due to the the `_handleDeposit` which is called in all of the above uses. 
`_handleDeposit` calls the `USDT` margin asset `approve` function with `type(uint).max`.
From the second time `approve` will be called, the transaction will revert.

`_handleDeposit` in `Trading`:
https://github.com/code-423n4/2022-12-tigris/blob/588c84b7bb354d20cbca6034544c4faa46e6a80e/contracts/Trading.sol#L652
```
    function _handleDeposit(address _tigAsset, address _marginAsset, uint256 _margin, address _stableVault
, ERC20PermitData calldata _permitData, address _trader) internal {
------
            IERC20(_marginAsset).transferFrom(_trader, address(this), _margin/_marginDecMultiplier);
            IERC20(_marginAsset).approve(_stableVault, type(uint).max);
            IStableVault(_stableVault).deposit(_marginAsset, _margin/_marginDecMultiplier);
------
    }
```

## Tools Used

VS Code

## Recommended Mitigation Steps

No need to to approve `USDT` every time. 
The protocol could:
1. Keep a record if allowance was already set on an address
2. Create an external function that can be called by the owner to approve the a token address