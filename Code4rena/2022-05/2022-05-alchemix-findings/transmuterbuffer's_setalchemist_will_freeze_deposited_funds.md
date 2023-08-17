## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [TransmuterBuffer's setAlchemist will freeze deposited funds](https://github.com/code-423n4/2022-05-alchemix-findings/issues/117) 

# Lines of code

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/TransmuterBuffer.sol#L230-L248


# Vulnerability details

Currently setAlchemist doesn't check whether there are any open positions left with the old Alchemist before switching to the new one.

As this require a number of checks the probability of operational mistake isn't low and it's prudent to introduce the main controls directly to the code to minimize it. In the case if the system go on with new Alchemist before realizing that there are some funds left in the old one, tedious and error prone manual recovery will be needed. There is also going to be a corresponding reputational damage.

Setting the severity to medium as while the function is admin only, the impact is up to massive user fund freeze, i.e. this is system breaking with external assumptions.

## Proof of Concept

Alchemist implementation change can happen while there are open deposits remaining with the current contract. As there looks to be no process to transfer them in the code, such TransmuterBuffer's funds will be frozen with old alchemist:

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-hardhat/TransmuterBuffer.sol#L230-L232

```solidity
    function setAlchemist(address _alchemist) external override onlyAdmin {
        sources[alchemist] = false;
        sources[_alchemist] = true;
```

## Recommended Mitigation Steps

Consider requiring that all exposure to the old Alchemist is closed, for example both `getAvailableFlow` and `getTotalCredit` is zero.

https://github.com/code-423n4/2022-05-alchemix/blob/de65c34c7b6e4e94662bf508e214dcbf327984f4/contracts-full/TransmuterBuffer.sol#L230-L231

```solidity
    function setAlchemist(address _alchemist) external override onlyAdmin {
+		  require(getTotalCredit() == 0, "Credit exists with old Alchemist");
+       for (uint256 j = 0; j < registeredUnderlyings.length; j++) {
+           require(getTotalUnderlyingBuffered[registeredUnderlyings[j]] == 0, "Buffer exists with old Alchemist");
+       }
        sources[alchemist] = false;
```

