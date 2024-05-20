# NFTs Are Not Random

## Severity
Medium Risk

## Relevant GitHub Links
https://github.com/Cyfrin/2024-05-Mondrian-Wallet/blob/main/contracts/MondrianWallet.sol

## Summary
The `MondrianWallet` contract suffers from a lack of randomness in assigning art to non-fungible tokens (NFTs). The current method of using the `token ID` to determine art assignment makes the process predictable, allowing users to foresee which art they will receive.

## Vulnerability Details
The vulnerability arises from the `tokenURI function`, which uses the `token ID` to determine art assignment. This approach makes the art assignment process predictable, as users can determine which art they will receive based on the `token ID`. The lack of randomness undermines the security and fairness of the NFT distribution process.

## PoC
To prove the lack of randomness in the `MondrianWallet` contract, we introduce a dummy `mint` function that increments a token ID counter, then write a test to mint a series of NFTs and verify the assigned art URIs.

```
uint256 i;
function mint() external {
    _mint(msg.sender, i);
    i++;
}
```
The test in JavaScript using the Hardhat framework demonstrates the predictability of the art assignment.

```
it("Distribution and randomness is bad", async function () {
    let user1 = await ethers.getSigners();
    const numIterations = 10;
    for (let i = 0; i < numIterations; i++) {
        await mondrianWallet.mint();
    }

    // Assert the token URIs for the first 10 tokens
    assert.equal(await mondrianWallet.tokenURI(0), "ar://jMRC4pksxwYIgi6vIBsMKXh3Sq0dfFFghSEqrchd_nQ");
    assert.equal(await mondrianWallet.tokenURI(1), "ar://8NI8_fZSi2JyiqSTkIBDVWRGmHCwqHT0qn4QwF9hnPU");
    assert.equal(await mondrianWallet.tokenURI(2), "ar://AVwp_mWsxZO7yZ6Sf3nrsoJhVnJppN02-cbXbFpdOME");
    assert.equal(await mondrianWallet.tokenURI(3), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
    assert.equal(await mondrianWallet.tokenURI(4), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
    assert.equal(await mondrianWallet.tokenURI(5), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
    assert.equal(await mondrianWallet.tokenURI(6), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
    assert.equal(await mondrianWallet.tokenURI(7), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
    assert.equal(await mondrianWallet.tokenURI(8), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
    assert.equal(await mondrianWallet.tokenURI(9), "ar://n17SzjtRkcbHWzcPnm0UU6w1Af5N1p0LAcRUMNP-LiM");
});
```
## Impact
The predictability of art assignment allows users to manipulate the minting process to obtain specific art pieces, undermining the fairness and randomness expected in the NFT distribution. This will lead to a loss of trust in the protocol and dissatisfaction among users.

## Tools Used

1. Manual code review
2. Hardhat

## Recommendations
To improve the randomness and fairness of the art assignment, the contract should use a more secure source of randomness. Consider using `RANDOMIZER.AI`, which provides robust and secure randomness. This solution works on both `Ethereum` and `ZkSync`, ensuring a consistent and fair distribution of art assignments. By integrating `RANDOMIZER.AI`, the assignment of art pieces to tokens can be reliably randomized at the time of minting, improving the overall security and functionality of the NFT distribution process. To interact with `RANDOMIZER.AI`, we will add the following interface:

```
interface IRandomizer {
    function request(uint256 callbackGasLimit) external returns (uint256);
    function request(uint256 callbackGasLimit, uint256 confirmations) external returns (uint256);
    function clientWithdrawTo(address _to, uint256 _amount) external;
}
```
To enhance the randomness of the assigned art, the following code should be added to the contract. The `_assignArt` function should be called at the time of minting to assign art to the token, ensuring each token receives an art piece immediately upon creation. The assigned art should be stored in a `mapping(uint256 => string)` called `IdAndItsArt`, guaranteeing consistent and retrievable art for each `token ID`. The `tokenURI` function should be updated to retrieve the assigned art from this mapping, making it deterministic and ensuring it always returns the correct art.

```
constructor(address entryPoint, address randomizer) Ownable(msg.sender) ERC721("MondrianWallet", "MW") {
    i_entryPoint = IEntryPoint(entryPoint);

    // The address for randomizer will depend on the network we want to work on.
    i_randomizer = IRandomizer(randomizer);
}

IRandomizer private immutable i_randomizer;
mapping(uint256=>string) public IdAndItsArt;
uint256 random;

function _assignArt(uint256 tokenId) internal {
    // Request a random value from RANDOMIZER.AI
    i_randomizer.request(50000);

    // Based on the generated random value, assign an art piece to the tokenId.
    if (random == 0) {
        IdAndItsArt[tokenId] = ART_ONE;
    } else if (random == 1) {
        IdAndItsArt[tokenId] = ART_TWO;
    } else if (random == 2) {
        IdAndItsArt[tokenId] = ART_THREE;
    } else {
        IdAndItsArt[tokenId] = ART_FOUR;
    }
}

function tokenURI(uint256 tokenId) public view override returns (string memory) {
    if (ownerOf(tokenId) == address(0)) {
        revert MondrainWallet__InvalidTokenId();
    }

    // Return the art assigned to the tokenId.
    return IdAndItsArt[tokenId];
}

function randomizerCallback(uint256 _id, bytes32 _value) external {
    // Callback can only be called by randomizer
    require(msg.sender == address(i_randomizer), "Caller not Randomizer");

    // Calculate the random value using modulo 4 to ensure fair distribution
    random = uint256(_value) % 4;
}

// Withdraw funds deposited with RANDOMIZER.AI 
function randomizerWithdraw(uint256 amount) external onlyOwner {
    i_randomizer.clientWithdrawTo(msg.sender, amount);
}

```
By generating a random value and performing an operation `% 4` on the random value, we solve the issue of bad randomness and ensure a secure, fair, and predictable assignment of art to each minted token.