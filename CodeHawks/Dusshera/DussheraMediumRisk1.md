# Function  `increaseValuesOfParticipants` Allows Organiser to Retain Control Over Ram Selection in `ChoosingRam` Contract            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-06-Dussehra/blob/main/src/ChoosingRam.sol

## Summary
The increaseValuesOfParticipants function in the ChoosingRam contract contains a vulnerability that fails to update the isRamSelected variable upon selecting a Ram. This oversight grants the organiser continuous control over the selection process, as only the selectRamIfNotSelected function can change the isRamSelected variable to true. This vulnerability undermines the fairness of the Ram selection process and allows the organiser to manipulate the protocol.

## Vulnerability Details
The vulnerability in the `increaseValuesOfParticipants` function arises from the failure to update the `isRamSelected` variable when a` Ram` is selected. This oversight allows the `organiser` to perpetually control the `Ram` selection process, undermining fairness and transparency. The flaw enables the `organiser` to exploit the selection mechanism by ensuring that only they can finalize the `Ram` selection, regardless of the initial selections made by `participants`. As a result, the vulnerability significantly compromises the integrity of the `Dussehra` event and exposes participants to unfair treatment.
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
The vulnerability arises from a logic flaw in the `increaseValuesOfParticipants` function, which neglects to update the `isRamSelected` variable upon selecting a `Ram`. This omission grants the `organiser` ongoing control over the selection process, enabling manipulation and compromising the event's fairness.

##### Likelihood of Occurrence
This vulnerability is highly likely to be exploited, as it provides a clear and continuous advantage to the organiser. Given the organiser's knowledge of the contract and their ability to manipulate the selection process, they have a strong incentive to exploit this flaw.

## Proof of Code
The `PoC` reveals a flaw in the `increaseValuesOfParticipants` function, impacting the `Ram` selection process in the `Dussehra` event. Here's how the test unfolds:

1. The participant deposit `1 ether`. `Player1` triggers` increaseValuesOfParticipants` five times in a row, passing his `token ID` as both `challenge`r and `participant` arguments to get selected as `Ram` (shortcut). Despite being selected, `isRamSelected` remains `false`. The `organiser` then exploits the flaw, setting` isRamSelected` to true. 

```solidity
function test_ifOrganiserRetainsControlOverRamSelection() external {
    vm.startPrank(player1);
    vm.deal(player1, 1 ether);
    dussehra.enterPeopleWhoLikeRam{value: 1 ether}();
    vm.stopPrank();
    
    vm.startPrank(player1);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);
    choosingRam.increaseValuesOfParticipants(0, 0);

    // Assuming increaseValuesOfParticipants selects Ram but does not set isRamSelected to true
    address selectedRam = choosingRam.selectedRam();
    bool isRamSelected = choosingRam.isRamSelected();
    assertEq(isRamSelected, false);
    assertEq(selectedRam, player1);

    vm.stopPrank();

    
    vm.warp(1728691200);

    vm.startPrank(organiser);
    choosingRam.selectRamIfNotSelected();

    assertEq(choosingRam.isRamSelected(), true);

    
}
```
After running the test, the test confirms that `isRamSelected` is now `true`, but only because the `organiser` explicitly called the function that changes its state.
```
forge test --match-test test_ifOrganiserRetainsControlOverRamSelection 
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[PASS] test_ifOrganiserRetainsControlOverRamSelection() (gas: 312980)
Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.66ms (589.83µs CPU time)

Ran 1 test suite in 3.49ms (1.66ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
This `PoC` clearly illustrates that the `organiser` can manipulate the selection process, retaining control over it by exploiting the vulnerability in the `increaseValuesOfParticipants` function.

## Impact
The vulnerability in the `increaseValuesOfParticipants` function poses a risk to the integrity and fairness of the `Ram` selection process in the `Dussehra` event. Its implications are profound:

1. `Organiser Control`: The vulnerability grants the `organiser` undue influence over the `Ram` selection, compromising fairness.

2. `Participant Disadvantage`: `Participants` face an unfair selection process, eroding trust and potentially discouraging future involvement.

3. `Protocol Integrity`: The vulnerability undermines the reliability of the `Dussehra` protocol, impacting stakeholder confidence.

4. `Trust Erosion`: Overall, the vulnerability undermines trust in the event's governance, necessitating swift mitigation measures.

## Tools Used
1. Manual Code Review
2. Foundry

## Recommendations
To fix this vulnerability, the `increaseValuesOfParticipants` function should update the` isRamSelected` variable to `true` when a `Ram` is selected. This ensures that once a `Ram` is selected, the `organiser` cannot exploit the selection process.
By implementing these changes, the integrity of the `Dussehra` event will be preserved, and `participant`s can trust that the selection process is fair and transparent. Properly managing the state variables will ensure that the `organiser` cannot manipulate the protocol for their own benefit.
```diff
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
+              isRamSelected= true;
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
+             isRamSelected= true;
            }
        }
    }
```
After rerunning the `test`, we observe that the test now` fails` because the `Ram` is already selected, indicating that the vulnerability has been successfully addressed.
slight change in assertion ive put isramselected, true;
```
forge test --match-test test_ifOrganiserRetainsControlOverRamSelection -vv
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[FAIL. Reason: revert: Ram is selected!] test_ifOrganiserRetainsControlOverRamSelection() (gas: 304994)
Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 1.60ms (515.02µs CPU time)

Ran 1 test suite in 3.28ms (1.60ms CPU time): 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/Dussehra.t.sol:CounterTest
[FAIL. Reason: revert: Ram is selected!] test_ifOrganiserRetainsControlOverRamSelection() (gas: 304994)

Encountered a total of 1 failing tests, 0 tests succeeded
```
## Conclusion:
In conclusion, fixing the vulnerability in the `increaseValuesOfParticipants` function has restored the integrity and fairness of the `Ram` selection process in the `Dussehra` event. By ensuring that the `isRamSelected` variable is updated appropriately, `participants` can trust that the selection process is transparent and free from manipulation, enhancing the overall credibility of the event.