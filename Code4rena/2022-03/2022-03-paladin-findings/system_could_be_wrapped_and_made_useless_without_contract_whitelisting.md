## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [System could be wrapped and made useless without contract whitelisting](https://github.com/code-423n4/2022-03-paladin-findings/issues/77) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L253
https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L284
https://github.com/code-423n4/2022-03-paladin/blob/9c26ec8556298fb1dc3cf71f471aadad3a5c74a0/contracts/HolyPaladinToken.sol#L268


# Vulnerability details

## Impact
Anyone could create a contract or a contract factory "PAL Locker" with a fonction to deposit PAL tokens through a contract, lock them and delegate the voting power to the contract owner. Then, the ownership of this contract could be sold. By doing so, locked hPAL would be made liquid and transferrable again. This would eventually break the overall system of hPAL, where the idea is that you have to lock them to make them non liquid to get a boosted voting power and reward rate. 

Paladin should expect this behavior to happen as we've seen it happening with veToken models and model implying locking features (see https://lockers.stakedao.org/ and https://www.convexfinance.com/). 

This behavior could eventually be beneficial to the original DAO (ex. https://www.convexfinance.com/ for Curve and Frax), but the original DAO needs to at least be able to blacklist / whitelist such contracts and actors to ensure their interests are aligned with the protocol.

## Proof of Concept

To make locked hPAL liquid, Alice could create a contact C. Then, she can deposit hPAL through the contract, lock them and delegate voting power to herself. She can then sell or tokenize the ownership of the contract C.

## Recommended Mitigation Steps

Depending of if Paladin wants to be optimistic or pessimistic, implement a whitelisting / blacklisting system for contracts. 

See:
https://github.com/curvefi/curve-dao-contracts/blob/4e428823c8ae9c0f8a669d796006fade11edb141/contracts/VotingEscrow.vy#L185

https://github.com/FraxFinance/frax-solidity/blob/7375949a73042c1e6dd14848fc4ea1ba62e36fb5/src/hardhat/contracts/FXS/veFXS_Solidity.sol.old#L370

