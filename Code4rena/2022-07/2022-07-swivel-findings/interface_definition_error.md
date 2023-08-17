## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Interface definition error](https://github.com/code-423n4/2022-07-swivel-findings/issues/39) 

# Lines of code

https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Marketplace/Interfaces.sol#L52
https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Marketplace/MarketPlace.sol#L164
https://github.com/code-423n4/2022-07-swivel/blob/fd36ce96b46943026cb2dfcb76dfa3f884f51c18/Swivel/Swivel.sol#L620
https://github.com/Swivel-Finance/gost/blob/a76ac859df049527c3e5df85e706dec6ffa0e2bb/test/swivel/Swivel.sol#L10


# Vulnerability details

## Impact
MarketPlace.authRedeem() call interface ISwivel.authRedeem() but Swivel contract  does not have this method only method  "authRedeemZcToken()"
The result will cause MarketPlace.authRedeem() to fail forever, thus causing ZcToken.withdraw() to fail forever

## Proof of Concept


MarketPlace.sol call ISwivel.authRedeem() 
```
  function authRedeem(uint8 p, address u, uint256 m, address f, address t, uint256 a) public authorized(markets[p][u][m].zcToken) returns (uint256 underlyingAmount) {
     .....

      ISwivel(swivel).authRedeem(p, u, market.cTokenAddr, t, a);

     .....
    } else {

     .....
      ISwivel(swivel).authRedeem(p, u, market.cTokenAddr, t, amount);
     ....
    }
```

Swivel.sol does not have   authRedeem() ,only authRedeemZcToken()
```
  function authRedeemZcToken(uint8 p, address u, address c, address t, uint256 a) external authorized(marketPlace) returns(bool) {
    // redeem underlying from compounding
    if (!withdraw(p, u, c, a)) { revert Exception(7, 0, 0, address(0), address(0)); }
    // transfer underlying back to msg.sender
    Safe.transfer(IErc20(u), t, a);

    return (true);
  }

```

## Tools Used

## Recommended Mitigation Steps

Swivel contract need declare "is ISwivel" and change method name
Other contracts should also declare "is Iinterfacename" to avoid method name errors
like IMarketPlace
 

