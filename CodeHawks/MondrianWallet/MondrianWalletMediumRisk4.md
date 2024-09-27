# Users unable to `mint` NFTs

## Severity
Medium Risk

## Relevant GitHub Links
	
https://github.com/Cyfrin/2024-05-Mondrian-Wallet/blob/main/contracts/MondrianWallet.sol

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol

## Summary
This submission details a critical vulnerability in the `MondrianWallet` smart contract, specifically related to the inability of users to mint NFTs. The issue arises due to the absence of a function dedicated to the minting process within the contract. As a result, users are unable to generate and own the NFTs promised by the` MondrianWallet`.

## Vulnerability Details
The `MondrianWallet` smart contract, designed to offer users a unique account abstraction wallet combined with random Mondrian art NFTs, lacks a crucial functionality, the ability to `mint` NFTs. The code provided does not include a method to facilitate the minting process, preventing users from obtaining the NFTs that are an integral part of the project’s value proposition.

## PoC

To address the vulnerability where users are unable to mint NFTs in the `MondrianWallet` contract, we conducted a comprehensive analysis of the entire contract and its functions. Our examination revealed the following critical insights:
1. Examination of the `MondrianWallet` contract confirms that the `_mint` and `_safeMint` functions are internal.
2. No external functions exist in the contract that would enable users to trigger minting.
3. Attempting to call the `_mint` or `_safeMint` functions externally results in a compilation error due to their internal visibility.
```solidity
function _mint(address to, uint256 tokenId) internal {
        if (to == address(0)) {
            revert ERC721InvalidReceiver(address(0));
        }
        address previousOwner = _update(to, tokenId, address(0));
        if (previousOwner != address(0)) {
            revert ERC721InvalidSender(address(0));
        }
    }

    function _safeMint(address to, uint256 tokenId, bytes memory data) internal virtual {
        _mint(to, tokenId);
        ERC721Utils.checkOnERC721Received(_msgSender(), address(0), to, tokenId, data);
    }
```
These tests aim to verify whether it is possible to mint NFTs by attempting to call the internal `_mint` and `_safeMint` functions externally. 
```javascript
it("testThatUsersCantMint", async function () {
        //dummy address
        const userAddress = "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4";
        await mondrianWallet._mint(userAddress, 1);
    })
    
    it("testThatUsersCantSafeMint", async function () {
        //dummy address
        const userAddress = "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4";
        await mondrianWallet._safeMint(userAddress, 2, bytes(""));
    })
```
However, since these functions are defined as `internal` within the contract, they cannot be accessed externally, leading to compilation errors during testing. The encountered errors indicate that the functions are not callable externally, resulting in the test failures.
```
 1) MondrianWallet
       testThatUsersCantMint:
     TypeError: mondrianWallet._mint is not a function
      at Context.<anonymous> (test/MondrianWallet.test.js:60:30)

  2) MondrianWallet
       testThatUsersCantSafeMint:
     TypeError: mondrianWallet._safeMint is not a function
      at Context.<anonymous> (test/MondrianWallet.test.js:65:30)
```
## Impact
The absence of the ability to `mint` NFTs directly impacts the interaction between users and the `MondrianWallet` protocol. Users are unable to obtain the expected NFTs, which will result in a loss of trust and engagement with the platform.

## Tools Used
1. Manual code review. 
2. Hardhat

## Recommendations
To resolve the issue of users being unable to mint NFTs in the `MondrianWallet` smart contract, it is recommended to implement the internal `_mint` function within the contract. This can be done by either creating a new public or external function that utilizes` _mint`, or by integrating` _mint` into an existing function, depending on the desired functionality of the code. Additionally, a new variable, such as `tokenIdCounter`, should be introduced to keep track of the tokens minted and automatically increment after each mint. It is also essential to maintain a record of the users who have already minted an NFT to ensure accurate tracking and prevent duplicate minting.