## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-07-fractional-findings/issues/609) 

# Table of contents

- **[[0x0] Disclaimer](#0x0)**
- **[[G-01] Try ++i instead of i++](G-01)**
- **[[G-02] Try `unchecked{++i}` instead of `i++` in loops](G-02)**
- **[[G-03] Consider `a = a + b` instead of `a += b`](G-03)**
- **[[G-04] Consider marking onlyOwner functions as payable](G-04)**
- **[[G-05] Use binary shifting instead of `a / 2^x, x > 0`](G-05)**
- **[[G-06] Cache state variables, `MLOAD` << `SLOAD`](G-06)**
- **[[G-07] Declare `immutable` instead of state variables](G-07)**
- **[[G-08] Define `constants/immutable/state` as `private/internal`](G-08)**
- **[[G-09] Check out `calldataloud` vs `mload`](G-09)**
- **[[G-10] `Internal` functions can be inlined](G-10)**
- **[[G-11] Functions are invoked inside the SC should be marked as internal](G-11)**
- **[[G-12] Redundant gas usage](G-12)**
- **[[G-13] Remove unnecessary explicit casts](G-13)**



## Disclaimer<a name="0x0"></a>
- Please, consider everything described below as a general recommendation. These notes will represent potential possibilities to optimize gas consumption. It's okay, if something is not suitable in your case. Just let me know the reason in the comments section. Enjoy!


## **[G-01] Try ++i instead of i++**<a name="G-01"></a>

### ***Description:***
  - In case of i++, the compiler needs to create a temp variable to return and then it gets incremented.  
  - In case of ++i, the compiler just simply returns already incremented value.

### ***Recommendations:***
  - Use prefix increment instead of postfix. 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/Vault.sol
      ...............................
      
        // Lines: [78-78]
        for (uint256 i = 0; i < length; i++) {}

        // Lines: [104-104]
        for (uint256 i = 0; i < length; i++) {}

    ```

## **[G-02] Try `unchecked{++i};` instead of `i++;` in loops**<a name="G-02"></a>

### ***Description:***
  - If the for loop runs 100 times, it's about 10k units of gas which can be saved in comparison. Don't worry about overflow, when the number is just simply getting incremented by 1. There are ~1e80 atoms in the universe, so 2^256 is closed to that number, therefore it's no a way to be overflowed, because of the gas limit as well.   

### ***Recommendations:***
  - Try to use unchecked{} box where it's no a way to get a overflow/underflow. Significant gas usage optimization.

### ***All occurances:***

  - Contracts:

    ```Solidity
      file: src/Vault.sol
      ...............................
      
        // Lines: [78-78]
          for (uint256 i = 0; i < length; i++) {}

        // Lines: [104-104]
          for (uint256 i = 0; i < length; i++) {}

    ```
## **[G-03] Consider `a = a + b` instead of `a += b`**<a name="G-03"></a>

### ***Description:***
  - It has an impact on the deployment cost and the cost for distinct transaction. 


### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [86-86]
          totalSupply[_id] += _amount;

        // Lines: [270-271]
          balanceOf[_from][_id] -= _amount;
          balanceOf[_to][_id] += _amount;

      file: src/Buyout.sol
      ...............................

        // Lines: [176-176]
          buyoutInfo[_vault].ethBalance += msg.value;

        // Lines: [139-139]
          buyoutInfo[_vault].ethBalance -= ethAmount;

      file: src/Migration.sol
      ...............................

        // Lines: [123-124]
          proposal.totalEth += msg.value;
          userProposalEth[_proposalId][msg.sender] += msg.value;

        // Lines: [134-135]
          proposal.totalFractions += _amount;
          userProposalFractions[_proposalId][msg.sender] += _amount;
          
        // Lines: [156-156]
          proposal.totalFractions -= amount;

        // Lines: [160-160]
          proposal.totalEth -= ethAmount;

        // Lines: [497-497]
          treeLength += IModule(_modules[i]).getLeafNodes().length;

      file: src/MerkleBase.sol
      ...............................

        // Lines: [147-147]
          for (uint256 i; i < length - 1; i += 2) {}

        // Lines: [190-190]
          ceil -= pOf2; // see above


## **[G-04] Consider marking onlyOwner functions as payable**<a name="G-04"></a>

### ***Description:***
  - A little optmization in comparison between payable and non-payable functions. Also, there is a little tradeoff here between readability and optimization.

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [56-63]
          function burn(
              address _from,
              uint256 _id,
              uint256 _amount
          ) external onlyRegistry {}

        // Lines: [79-87]
          function mint(
              address _to,
              uint256 _id,
              uint256 _amount,
              bytes memory _data
          ) external onlyRegistry {}

        // Lines: [198-200]
          function setContractURI(string calldata _uri) external onlyController {}

        // Lines: [205-211]
          function setMetadata(address _metadata, uint256 _id)
              external
              onlyController
          {}

        // Lines: [217-225]
          function setRoyalties(
              uint256 _id,
              address _receiver,
              uint256 _percentage
          ) external onlyController {}

        // Lines: [229-232]
          function transferController(address _newController)
              external
              onlyController
          {}
        ```
## **[G-05] Use binary shifting instead of `a / 2^x, x > 0`**<a name="G-05"></a>

### ***Description:***
  - It's also pretty impactful approach especially in loops. 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/MerkleBase.sol
      ...............................
      
        // Lines: [100-100]
          _node = _node / 2;

        // Lines: [136-136]
          result = new bytes32[](length / 2 + 1);

        // Lines: [142-142]
          result = new bytes32[](length / 2);
      ```
## **[G-06] Cache state variables, `MLOAD` << `SLOAD`**<a name="G-06"></a>

### ***Description:***
  - `MLOAD` costs only 3 units of gas, `SLOAD`(warm access) is about 100 units. Therefore, cache, when it's possible. 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................

        // Lines: [91-91]
          if (msg.sender != metadata[_id])
            revert InvalidSender(metadata[_id], msg.sender); 

        // Lines: [297-298]
          if (msg.sender != metadata[_id])
            revert InvalidSender(metadata[_id], msg.sender);

        // Lines: [303-305]
          _controller == address(0)
              ? controllerAddress = INITIAL_CONTROLLER()
              : controllerAddress = _controller;

      file: src/Bayout.sol
      ...............................
        // Lines: [176-176]
          buyoutInfo[_vault].ethBalance += msg.value;


      file: src/Vault.sol
      ...............................

        // Lines: [25-25]
          if (nonce != 0) revert Initialized(owner, msg.sender, nonce);

        // Lines: [76, 87, 94, 102]
          if (owner != msg.sender) revert NotOwner(owner, msg.sender);

        // Lines: [132-132]
          if (owner_ != owner) revert OwnerChanged(owner_, owner);

      file: src/Migration.sol
      ...............................

        // Lines: [126-127; 137-138]
          // Comment:
            - proposal.totalEth += msg.value => proposal.totalEth = proposal.totalEth(SLOAD) + msg.value; 
  
             - proposal.totalEth = proposal.totalEth(which you can store in memory to avoid SLOAD) + msg.value. 

            proposal.totalEth += msg.value;
            userProposalEth[_proposalId][msg.sender] += msg.value;  

            proposal.totalFractions += _amount;
            userProposalFractions[_proposalId][msg.sender] += _amount;

        file: src/Buyout.sol
        ...............................
          // Lines: [474-502]
              permissions[0] = Permission(
              address(this),
              supply,
              ISupply(supply).burn.selector
            );
            permissions[1] = Permission(
              address(this),
              transfer,
              ITransfer(transfer).ERC20Transfer.selector
            );
            permissions[2] = Permission(
              address(this),
              transfer,
              ITransfer(transfer).ERC721TransferFrom.selector
            );
            permissions[3] = Permission(
              address(this),
              transfer,
              ITransfer(transfer).ERC1155TransferFrom.selector
            );
            permissions[4] = Permission(
              address(this),
              transfer,
              ITransfer(transfer).ERC1155BatchTransferFrom.selector
            );

        ```

## **[G-07] Declare `immutable` instead of state variables**<a name="G-07"></a>

### ***Description:***
  - Since it's initialized once, there is no reason for state variable allocation. 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [19-19]
          address internal _controller;

        // Lines: [21-21]
          string public contractURI;

        // Lines: [142-142]
          result = new bytes32[](length / 2);

      file: src/Vault.sol
      ...............................
        // Lines: [15-15]
          bytes32 public merkleRoot;

        // Lines: [17-17]
          uint256 public nonce;

      file: src/VaultFactory.sol
      ...............................
        // Lines: [15-15]
          address public implementation;

      file: src/Buyout.sol
      ...............................
        // Lines: [29-33]
          address public registry;
          address public supply;
          address public transfer;

      ```
## **[G-08] Define public `constants/immutable/state` as `private/internal`**<a name="G-08"></a>

### ***Description:***
  - Declaring state variables as private/internal doesn't generate getter functions. 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [15-17]
          string public constant NAME = "FERC1155";
          string public constant VERSION = "1";

        // Lines: [21-21]
          string public contractURI;

        // Lines: [23-30]
          mapping(address => mapping(address => mapping(uint256 => bool)))
        public isApproved;
          /// @notice Mapping of metadata contracts for token ID types => metadata address
          mapping(uint256 => address) public metadata;
          /// @notice Mapping to track account nonces for metadata txs owner => nonces
          mapping(address => uint256) public nonces;
          /// @notice Mapping to track total supply for token ID types => totalSupply
          mapping(uint256 => uint256) public totalSupply;

      file: src/Vault.sol
      ...............................
        // Lines: [21-21]
          mapping(bytes4 => address) public methods;

      file: src/VaultRegistry.sol
      ...............................
        // Lines: [17-21]
          address public immutable factory;
          address public immutable fNFT;
          address public immutable fNFTImplementation;

        // Lines: [23-25]
          mapping(address => uint256) public nextId;
          mapping(address => VaultInfo) public vaultToToken;

      file: src/Buyout.sol
      ...............................
        // Lines: [35-38]
          uint256 public constant PROPOSAL_PERIOD = 2 days;
          uint256 public constant REJECTION_PERIOD = 4 days;
          mapping(address => Auction) public buyoutInfo;
    ```
## **[G-09] Check out `calldataloud` vs `mload`**<a name="G-09"></a>

### ***Description:***
  - Consider reading args directly from calldata instead of memory, if args doesn't require any changes. 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [68-68]
          function emitSetURI(uint256 _id, string memory _uri) external {}

        // Lines: [79-79]
        // Comments: OZ marked `bytes memory _data`, it is because those functions are virtual, hence could be overrided. Therefore, allowing modifying args for those overrided versions. Here, we just have an external function, therefore it's better to read bytes directly from calldata, especially, if the `_data` is a massive flow. 

          function mint(
              address _to,
              uint256 _id,
              uint256 _amount,
              bytes memory _data
          ) external onlyRegistry {}

        // Lines: [68-68]
          function emitSetURI(uint256 _id, string memory _uri) external {}

      file: src/Vault.sol
      ...............................
      
        // Lines: [73-75]
          function install(bytes4[] memory _selectors, address[] memory _plugins)
              external
          {}
        // Lines: [101-101]
          function uninstall(bytes4[] memory _selectors) external {}

      file: src/VaultRegistry.sol
      ...............................
      
        // Lines: [51-54]
          function create(
              bytes32 _merkleRoot,
              address[] memory _plugins,
              bytes4[] memory _selectors
          ) external returns (address vault) {}

        // Lines: [67-72]
          function createFor(
              bytes32 _merkleRoot,
              address _owner,
              address[] memory _plugins,
              bytes4[] memory _selectors
          ) external returns (address vault) {}

        // Lines: [67-72]
          function createCollection(
            bytes32 _merkleRoot,
            address[] memory _plugins,
            bytes4[] memory _selectors
        ) external returns (address vault, address token) {}

        // Lines: [102-107]
          function createInCollection(
              bytes32 _merkleRoot,
              address _token,
              address[] memory _plugins,
              bytes4[] memory _selectors
          ) external returns (address vault) {}

        // Lines: [147-152]
          function createCollectionFor(
              bytes32 _merkleRoot,
              address _controller,
              address[] memory _plugins,
              bytes4[] memory _selectors
          ) public returns (address vault, address token) {}

        // Lines: [165-170]
          function _deployVault(
              bytes32 _merkleRoot,
              address _token,
              address[] memory _plugins,
              bytes4[] memory _selectors
          ) private returns (address vault) {}

      file: src/Buyout.sol
      ...............................
          function batchWithdrawERC1155(
              address _vault,
              address _token,
              address _to,
              uint256[] calldata _ids,
              uint256[] calldata _values,
              bytes32[] calldata _erc1155BatchTransferProof
          ) external {}

          // Look, how beatufil it looks like with calldata. Thank you for that!!!!!!!

    ```
## **[G-10] `Internal` functions can be inlined**<a name="G-10"></a>

### ***Description:***
  - It takes some extra `JUMP`s which costs around 12 gas uints for each `JUMP`. 
 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [324-330]
          function _computePermitStructHash(
              address _owner,
              address _operator,
              uint256 _id,
              bool _approved,
              uint256 _deadline
          ) internal returns (bytes32) {}

        // Lines: [350-355]
          function _computePermitAllStructHash(
              address _owner,
              address _operator,
              bool _approved,
              uint256 _deadline
          ) internal returns (bytes32) {}

        // Lines: [371-371]
          function _computeDomainSeparator() internal view returns (bytes32) {}

        // Lines: [388-392]
          function _computeDigest(bytes32 _domainSeparator, bytes32 _structHash)
              internal
              pure
              returns (bytes32)
          {}

    ```
## **[G-11] Functions which are invoked inside the SC should be marked as internal**<a name="G-11"></a>

### ***Description:***
  - If i'm not mistaken, these getter functions should be defined as internal. 
 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/FERC1155.sol
      ...............................
      
        // Lines: [309-316]
            function INITIAL_CONTROLLER() public pure returns (address) {
              return _getArgAddress(0);
            }

            function VAULT_REGISTRY() public pure returns (address) {
                return _getArgAddress(20);
            }
      ```
## **[G-12] Redundant gas usage**<a name="G-12"></a>

### ***Description:***
  - Extra gas usage without the reason, use _selectors.length in loops.
 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/Vault.sol
      ...............................
      
        // Lines: [77-77]
          uint256 length = _selectors.length;

        // Lines: [103-103]
          uint256 length = _selectors.length;

      ```
## **[G-13] Remove unnecessary explicit casts**<a name="G-13"></a>

### ***Description:***
  - There is no reason to explicitly cast `address` to `address`, etc... 
 

### ***All occurances:***

  - Contracts:
  
    ```Solidity
      file: src/VaultRegistry.sol
      ...............................
      
        // Lines: [73-73]
          vault = _deployVault(_merkleRoot, address(fNFT), _plugins, _selectors);

        // Lines: [56-56]
          vault = _deployVault(_merkleRoot, address(fNFT), _plugins, _selectors);

        // Lines: [154-154]
          abi.encodePacked(_controller, address(this))
      ```

## Kudos for the quality of the code! It's pretty easy to explore!
