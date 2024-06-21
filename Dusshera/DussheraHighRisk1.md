# Bad Randomness in `increaseValuesOfParticipants` and `selectRamIfNotSelected` functions            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-06-Dussehra/blob/main/src/ChoosingRam.sol

## Summary
The current implementation of the `ChoosingRam` contract contains a vulnerability in the randomness generation process within the `increaseValuesOfParticipants` and `selectRamIfNotSelected` functions. These functions utilize on-chain randomness generated using `block.timestamp` and` block.prevrandao` (formerly` block.difficulty`). While `block.prevrandao` provides a slightly more random value than `block.timestamp`, it still falls short of providing full randomness. This vulnerability undermines the fairness and integrity of the protocol, potentially leading to biased outcomes and loss of trust among participants.

## Vulnerability Details
The vulnerability arises from the use of on-chain randomness generation in the `increaseValuesOfParticipants` and `selectRamIfNotSelected` functions. These functions rely on `block.timestamp` and `block.prevrandao` for randomness, which can be predicted or influenced, allowing participants to manipulate outcomes in their favor. This undermines the fairness and integrity of the protocol, as randomness should be unpredictable and unbiased. While `block.prevrandao` offers slightly better randomness, it still lacks full unpredictability. 
```solidity
  function increaseValuesOfParticipants(uint256 tokenIdOfChallenger, uint256 tokenIdOfAnyPerticipent)
        public
        RamIsNotSelected
    {
        if (tokenIdOfChallenger > ramNFT.tokenCounter()) {
            revert ChoosingRam__InvalidTokenIdOfChallenger();
        }
        if (tokenIdOfAnyPerticipent > ramNFT.tokenCounter()) {
            revert ChoosingRam__InvalidTokenIdOfPerticipent();
        }
        if (ramNFT.getCharacteristics(tokenIdOfChallenger).ram != msg.sender) {
            revert ChoosingRam__CallerIsNotChallenger();
        }

        if (block.timestamp > 1728691200) {
            revert ChoosingRam__TimeToBeLikeRamFinish();
        }
        
        uint256 random =
            uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, msg.sender))) % 2;

        if (random == 0) {
            if (ramNFT.getCharacteristics(tokenIdOfChallenger).isJitaKrodhah == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, false, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isDhyutimaan == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isVidvaan == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isAatmavan == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isSatyavaakyah == false){
                ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, true);
                selectedRam = ramNFT.getCharacteristics(tokenIdOfChallenger).ram;
            }
        } else {
            if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isJitaKrodhah == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, false, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isDhyutimaan == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, false, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isVidvaan == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, false, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isAatmavan == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, true, false);
            } else if (ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).isSatyavaakyah == false){
                ramNFT.updateCharacteristics(tokenIdOfAnyPerticipent, true, true, true, true, true);
                selectedRam = ramNFT.getCharacteristics(tokenIdOfAnyPerticipent).ram;
            }
        }
    }

    function selectRamIfNotSelected() public RamIsNotSelected OnlyOrganiser {
        if (block.timestamp < 1728691200) {
            revert ChoosingRam__TimeToBeLikeRamIsNotFinish();
        }
        if (block.timestamp > 1728777600) {
            revert ChoosingRam__EventIsFinished();
        }
        uint256 random = uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao))) % ramNFT.tokenCounter();
        selectedRam = ramNFT.getCharacteristics(random).ram;
        isRamSelected = true;
    }
```
##### Cause of the Issue
The primary cause of the bad randomness vulnerability lies in the reliance on on-chain sources for generating random values. In the `Dussehra` event's `Ram` selection process, the function utilizes `block.prevranda`o to derive randomness. While `block.prevrandao` introduces some level of randomness, it is still deterministic and susceptible to manipulation. This reliance on on-chain sources for randomness compromises the integrity of the selection process, as it cannot guarantee truly random outcomes. 
```solidity
uint256 random =
            uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao, msg.sender))) % 2;
```
```solidity
uint256 random = uint256(keccak256(abi.encodePacked(block.timestamp, block.prevrandao))) % ramNFT.tokenCounter();
```

##### Explanation of block.prevrandao
1.` block.prevrandao` (formerly known as `block.difficulty`) is the most recent block's random value.
2. It is determined by miners and, although somewhat unpredictable, can be influenced by miners to a certain extent.
3. Combining `block.timestamp` and `block.prevrandao` with the `sender's address` provides some randomness, but this method is still susceptible to manipulation and prediction.

##### Likelihood of Exploitation
The likelihood of this vulnerability being exploited is high. Since the random value generation is predictable and can be influenced , the fairness and randomness of the selection process are compromised.

## Impact
1. `Unfair Advantage`: Allows for potential prediction and manipulation by miners or participants, undermining the fairness of the event.
2.  `Participant Disadvantage`: Honest participants are at a disadvantage due to the compromised random selection process, leading to a potential loss of trust in the event.
3.  `Protocol Integrity`: Compromises the integrity of the ChoosingRam protocol, making it susceptible to manipulation and abuse.

## Tools Used
1. Manual Code Review
2. Foundry

## Recommendations
To address the bad randomness vulnerability in the `ChoosingRam` protocol, we'll integrate the `Randomizer.AI` service for secure and reliable randomness generation, which provides robust and secure randomness. This solution works on all of the chains that this project will be deployed. Below are the steps and modifications necessary to implement this solution:



Detailed Code Changes
1. `Step 1`: Add the `Randomizer` Interface, this defines the functions we need to interact with the `Randomize`r service, including requesting randomness and withdrawing funds
```diff
+   interface IRandomizer {
+       function request(uint256 callbackGasLimit) external returns (uint256);
+       function request(uint256 callbackGasLimit, uint256 confirmations) external returns (uint256);
+       function clientWithdrawTo(address _to, uint256 _amount) external;
+   }
```
2. `Step 2`: Initialize `Randomize`r in the `Constructor`, update the contract's constructor to accept the `Randomizer` address and initialize it.

```diff
+   constructor(address _ramNFT, address randomizer) {
         isRamSelected = false;
         ramNFT = RamNFT(_ramNFT);
+       i_randomizer = IRandomizer(randomizer);
}

+I      Randomizer private immutable i_randomizer;
+       uint256 private random;
```
3. `Step 3`: Implement `Random Value Callback`, add the `randomizerCallback function` to handle the callback from `Randomizer`.

```diff
+ function randomizerCallback(uint256 _id, bytes32 _value) external {
+    require(msg.sender == address(i_randomizer), "Caller not Randomizer");
+    random = uint256(_value) ;
+ }
+
+ // Withdraw funds deposited with RANDOMIZER.AI
+ function randomizerWithdraw(uint256 amount) external onlyOwner {
+    i_randomizer.clientWithdrawTo(msg.sender, amount);
+}
```
4. `Step 4`: Modify Vulnerable Functions, `increaseValuesOfParticipants` and  `selectRamIfNotSelected` functions are updated to request randomness from `Randomize`r and use the received random value.
The random value is stored in the random state variable, which is then used to determine the outcome in each function.
```diff
function increaseValuesOfParticipants(uint256 tokenIdOfChallenger, uint256 tokenIdOfAnyParticipant)
    public
    RamIsNotSelected
    returns (uint256)
{
    if (tokenIdOfChallenger == tokenIdOfAnyParticipant) {
        revert ChoosingRam__SameTokenIdForBothChallengerAndParticipant();
    }
    if (tokenIdOfChallenger > ramNFT.tokenCounter()) {
        revert ChoosingRam__InvalidTokenIdOfChallenger();
    }
    if (tokenIdOfAnyParticipant > ramNFT.tokenCounter()) {
        revert ChoosingRam__InvalidTokenIdOfParticipant();
    }
    if (ramNFT.getCharacteristics(tokenIdOfChallenger).ram != msg.sender) {
        revert ChoosingRam__CallerIsNotChallenger();
    }

    if (block.timestamp > 1728691200) {
        revert ChoosingRam__TimeToBeLikeRamFinish();
    }

+    // Request randomness from Randomizer
+    i_randomizer.request(50000);
+    uint256 number = random % 2;

+    if (number == 0) {
        if (ramNFT.getCharacteristics(tokenIdOfChallenger).isJitaKrodhah == false){
            ramNFT.updateCharacteristics(tokenIdOfChallenger, true, false, false, false, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isDhyutimaan == false){
            ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, false, false, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isVidvaan == false){
            ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, false, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isAatmavan == false){
            ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfChallenger).isSatyavaakyah == false){
            ramNFT.updateCharacteristics(tokenIdOfChallenger, true, true, true, true, true);
            selectedRam = ramNFT.getCharacteristics(tokenIdOfChallenger).ram;
        }
    } else {
        if (ramNFT.getCharacteristics(tokenIdOfAnyParticipant).isJitaKrodhah == false){
            ramNFT.updateCharacteristics(tokenIdOfAnyParticipant, true, false, false, false, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfAnyParticipant).isDhyutimaan == false){
            ramNFT.updateCharacteristics(tokenIdOfAnyParticipant, true, true, false, false, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfAnyParticipant).isVidvaan == false){
            ramNFT.updateCharacteristics(tokenIdOfAnyParticipant, true, true, true, false, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfAnyParticipant).isAatmavan == false){
            ramNFT.updateCharacteristics(tokenIdOfAnyParticipant, true, true, true, true, false);
        } else if (ramNFT.getCharacteristics(tokenIdOfAnyParticipant).isSatyavaakyah == false){
            ramNFT.updateCharacteristics(tokenIdOfAnyParticipant, true, true, true, true, true);
            selectedRam = ramNFT.getCharacteristics(tokenIdOfAnyParticipant).ram;
        }
    }
}

function selectRamIfNotSelected() public RamIsNotSelected OnlyOrganiser {
    if (block.timestamp < 1728691200) {
        revert ChoosingRam__TimeToBeLikeRamIsNotFinish();
    }
    if (block.timestamp > 1728777600) {
        revert ChoosingRam__EventIsFinished();
    }

+    // Request randomness from Randomizer
+    i_randomizer.request(50000);
+    uint256 number = random % ramNFT.tokenCounter();

+    selectedRam = ramNFT.getCharacteristics(number).ram;
    isRamSelected = true;
}
```
By implementing these changes, the contract now uses a secure and reliable source of randomness, ensuring fairness and integrity in the participant selection process.