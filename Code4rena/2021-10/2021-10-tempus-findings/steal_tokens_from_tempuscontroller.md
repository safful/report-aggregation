## Tags

- bug
- sponsor confirmed
- disagree with severity
- 3 (High Risk)
- resolved

# [Steal tokens from TempusController](https://github.com/code-423n4/2021-10-tempus-findings/issues/10) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function _depositAndProvideLiquidity can be used go retrieve arbitrary ERC20 tokens from the TempusController.sol contract.

As the test contract of TempusController.sol https://goerli.etherscan.io/address/0xd4330638b87f97ec1605d7ec7d67ea1de5dd7aaa shows, it has indeed ERC20 tokens.

The problem is due to the fact that you supply an arbitrary tempusAMM to depositAndProvideLiquidity and thus to _depositAndProvideLiquidity. 
tempusAMM could be a fake contract that supplies values that are completely fake.

At the end of the function _depositAndProvideLiquidity, ERC20 tokens are send to the user. If you can manipulate the variables ammTokens,  mintedShares  and sharesUsed you can send back
any tokens held in the contract
"ammTokens[0].safeTransfer(msg.sender, mintedShares - sharesUsed[0]);"

The Proof of Concept shows an approach to do this. 


## Proof of Concept
https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusController.sol#L73-L79

https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/TempusController.sol#L304-L335

Create a fake Vault contract (fakeVault) with the following functions:
fakeVault.getPoolTokens(poolId) --> returns {TokenToSteal1,TokenToSteal2},{fakeBalance1,fakeBalance2},0
fakeVault.JoinPoolRequest() --> do nothing
fakeVault.joinPool() --> do nothing

Create a fake Pool contract (fakePool) with the following functions:
fakePool.yieldBearingToken() --> returns fakeYieldBearingToken
fakePool.deposit() --> returns fakeMintedShares,....

Create a fake ammTokens contract with the following functions:
tempusAMM.getVault() --> returns fakeVault
tempusAMM.getPoolId() --> returns 0
tempusAMM.tempusPool() --> returns fakePool


call depositAndProvideLiquidity(fakeTempusAMM,1,false) // false -> yieldBearingToken
_getAMMDetailsAndEnsureInitialized returns fakeVault,0, {token1,token2},{balance1,balance2}
_deposit(fakePool,1,false) calls _depositYieldBearing which calls fakePool.deposit()  and returns fakeMintedShares
_provideLiquidity(...)  calculates a vale of ammLiquidityProvisionAmounts
_provideLiquidity(...)  skips the safeTransferFrom because sender == address(this)) 
the calls to fakeVault.JoinPoolRequest() and fakeVault.joinPool() can be faked.
_provideLiquidity(...)  returns the value ammLiquidityProvisionAmounts

Now fakeMintedShares - ammLiquidityProvisionAmounts number of TokenToSteal1 and TokenToSteal2 are transferred to msg.sender

As you can both manipulate TokenToSteal1 and fakeMintedShares, you can transfer any token to msg.sender

## Tools Used

## Recommended Mitigation Steps
Create a whitelist for tempusAMMs


