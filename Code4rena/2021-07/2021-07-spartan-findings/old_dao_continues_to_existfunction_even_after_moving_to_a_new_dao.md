## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- disagree with severity

# [Old DAO continues to exist/function even after moving to a new DAO](https://github.com/code-423n4/2021-07-spartan-findings/issues/107) 

# Handle

0xRajeev


# Vulnerability details

## Impact

If moveDAO() is executed after voting, the existing DAO contract continues to function as before whereas it should ideally cease to function/exist from the users’ perspective or at least function as a clone of the new DAO by using the same addresses as it does.

Scenario: moveDAO is executed to make DAO and BASE.DAO point to the new address. Existing DAO contract continues to function but all the other interfacing contracts (ROUTER, UTILS, DAOVAULT, BONDVAULT, SYNTHVAULT, POOLFACTORY, SYNTHFACTORY and RESERVE) use the updated DAO address as updated in BASE. At a minimum, this leads to undefined behavior and at worst an attack where the old DAOs (there could be many) are exploited because it still points to valid router, pool and vault contracts.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L451-L459


changeDAO:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/outside-scope/Sparta.sol#L189-L193


Use of BASE.DAO:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/BondVault.sol#L54-L57

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/DaoVault.sol#L32-L34

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L39-L41

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L41-L43

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L20-L22

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Synth.sol#L20-L22

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Utils.sol#L29-L31

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L35-L37

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthFactory.sol#L27-L29

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/synthVault.sol#L45-L47

Updated Getters:
https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L614-L684

Example uses of stale interface contract addresses _* instead of using Dao(DAO).* versions:
_ROUTER: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L259

_BONDVAULT: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L281

_UTILS: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L205

_RESERVE: https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Dao.sol#L188

etc.

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

At a minimum, all DAO public/external functions should check and revert if daoHasMoved or the design can even consider a selfdestruct to destroy the DAO contract once it has successfully handed over to the new DAO contract and all pending actions have been cleared. In the unlikely requirement of older DAO contracts continuing to exist, they should at least use addresses of interfacing contracts as reported by the new DAO which could have updated them.

