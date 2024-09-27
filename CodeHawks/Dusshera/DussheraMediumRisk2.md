# `increaseValuesOfParticipants` Function Allows NFT `Owner` to Always Win            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-06-Dussehra/blob/main/src/ChoosingRam.sol

## Summary
The `increaseValuesOfParticipants` function in the `ChoosingRam` contract allows an `NFT`` owne`r to pass the same `tokenId` for both the `challenger` and any `participant`. This oversight results in the`NFT` `owner` winning every "battle", bypassing the intended randomness and fairness mechanisms. This vulnerability enables the `attacke` to manipulate the outcome and become the `selected Ram` with only five calls, undermining the integrity of the selection process.

## Vulnerability Details
The bug in the `increaseValuesOfParticipants` function allows a user to pass their `tokenId` as both the `challenger` and any` participant`, ensuring they win every "battle" and incrementally increase the value of their `NFT`. This exploit enables the user to manipulate the selection process and become the chosen `Ram` with just five calls, as each call updates one characteristic of their `NFT`.
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
```

##### Cause of the Issue
The vulnerability arises because the `increaseValuesOfParticipants` function does not check if the `tokenIdOfChallenger` and `tokenIdOfAnyPerticipent` arguments are the same. This omission allows an `NFT` `owne`r to always win by passing the same `tokenId` for both arguments, effectively bypassing the randomness intended by the function.

##### Likelihood of Exploitation

The likelihood of this vulnerability being exploited is relatively high. Since the function is publicly accessible and lacks any access restrictions, any user with knowledge of the contract can exploit it to always win the selection process by using the same `tokenId` for both arguments, thereby consistently increasing the value of their NFT and gaining an unfair advantage.

## Impact
The vulnerability in the` increaseValuesOfParticipants` function has significant implications for the `ChoosingRam` protocol, affecting fairness, participant experience, and the overall integrity of the event. Below are the specific impacts of this exploit:
1. `Unfair Advantage`: The exploit allows a user to gain an unfair advantage by always winning the selection process, which undermines the intended randomness and fairness of the event.
2. `Participant Disadvantage`: Honest participants are disadvantaged as they cannot compete fairly, leading to potential loss of trust in the event's integrity.
3. `Protocol Integrity`: The vulnerability compromises the overall integrity of the `ChoosingRam` protocol, making it susceptible to manipulation and abuse.

## Proof of Code
The proof of concept demonstrates that the function can be exploited by passing the same `tokenId` for both arguments, allowing the user to win every time and become the `selected Ram` after only five calls.
```solidity
function test_ifUserCanIncreaseOnlyTheValueOfHisNFT() external {
    vm.startPrank(player1);
    vm.deal(player1, 1 ether);
    dussehra.enterPeopleWhoLikeRam{value: 1 ether}();
    
    // Passing 0 (tokenId of player1) for both challenger and any participant
    // Ensuring player1 always wins and their NFT value always increases
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);

    vm.stopPrank();

    // Only 5 calls of the increaseValuesOfParticipants() are enough for player1 to get selected as Ram
    assertEq(choosingRam.selectedRam(), player1);
}
```
After running this `tes`t, the result confirms the existence of the vulnerability:
```
forge test --match-test test_ifUserCanIncreaseOnlyTheValueOfHisNFT -vvv
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[PASS] test_ifUserCanIncreaseOnlyTheValueOfHisNFT() (gas: 295872)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.70ms (600.44µs CPU time)

Ran 1 test suite in 3.61ms (1.70ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
This output indicates that the vulnerability is present and exploitable, allowing the `NFT owner` to always win and unfairly become the `selected Ram`.

## Tools Used
1. Manual Code Review
2. Foundry

## Recommendations
To fix this vulnerability, the` increaseValuesOfParticipants` function should be modified to check that `tokenIdOfChallenger` and `tokenIdOfAnyPerticipent `are not the same, also we added an `error` `ChoosingRam__SameTokenIdForBothChallengerAndParticipant`. This ensures that a user cannot exploit the function by passing the same `tokenId` for both arguments.
```diff
+      error ChoosingRam__SameTokenIdForBothChallengerAndParticipant();

  function increaseValuesOfParticipants(uint256 tokenIdOfChallenger, uint256 tokenIdOfAnyPerticipent)
        public
        RamIsNotSelected
    {
+      if (tokenIdOfChallenger == tokenIdOfAnyPerticipent) {
+          revert ChoosingRam__SameTokenIdForBothChallengerAndParticipant();
+      }
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
```
After updating the code, we are running the `test` again:
```
forge test --match-test test_ifUserCanIncreaseOnlyTheValueOfHisNFT -vv
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[FAIL. Reason: ChoosingRam__SameTokenIdForBothChallengerAndParticipant()] test_ifUserCanIncreaseOnlyTheValueOfHisNFT() (gas: 194460)
Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 1.16ms (71.38µs CPU time)

Ran 1 test suite in 2.72ms (1.16ms CPU time): 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/Dussehra.t.sol:CounterTest
[FAIL. Reason: ChoosingRam__SameTokenIdForBothChallengerAndParticipant()] test_ifUserCanIncreaseOnlyTheValueOfHisNFT() (gas: 194460)

Encountered a total of 1 failing tests, 0 tests succeeded
```
The test failed, indicating that the vulnerability has been successfully mitigated.
## Conclusion
Addressing this vulnerability ensures that the selection process remains fair and prevents any participant from exploiting the function to gain an advantage. Implementing the recommended fix will enhance the protocol's integrity and maintain participant trust in the `ChoosingRam` event.