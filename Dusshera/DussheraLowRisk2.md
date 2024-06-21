# Incorrect Timestamp Validation in `killRavana` Function of `Dussehra` Contract            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-06-Dussehra/blob/main/src/Dussehra.sol

## Summary
The `killRavana` function in the `Dussehra` contract contains incorrect `timestamp` checks, allowing the function to be executed outside the intended window of `12th October 2024` to `13th October 2024`. This could lead to unintended and potentially harmful executions of the function, disrupting the event's integrity.

## Vulnerability Details
The `killRavana` function is designed to be executed within a specific time frame: after `12th October 2024` and before `13th October 2024`. However, the current implementation uses incorrect timestamp values, allowing the function to be called `one minute` before `12th October 2024` and `one minute` after `13th October 2024`. Specifically, the timestamps `1728691029` and `1728777669` are not accurate representations of the intended dates.

##### Cause of the Issue
The incorrect `timestamp` values in the code cause this issue. The values `1728691029` and `1728777669` do not align with the exact start and end times for `12th October 2024` and `13th October 2024`, respectively.
The timestamps used in the `killRavana` function translate to the: 
1. `1728691069` == `Fri Oct 11. 2024. 23:57:49 GMT+0000 `
2. `1728777669` == `Sun, Oct 13. 2024. 00:01:09 GMT+0000` 

##### Likelihood of Occurrence
This issue is very likely to occur as it directly stems from hardcoded incorrect `timestamps`. Any user attempting to execute the `killRavana` function within the specified range but slightly off by a minute will encounter this bug.
## Proof of Code
This `PoC` demonstrates that the `killRavana` function can be executed one minute after the intended window, showing that the incorrect `timestamp` values allow the function to be called outside the designated period.
```solidity
function test_killRavanaAfter13thOfOctober()external{
         vm.startPrank(player1);
        vm.deal(player1, 1 ether);
        dussehra.enterPeopleWhoLikeRam{value: 1 ether}();
        vm.stopPrank();
        
        vm.warp(1728691200 + 1);
        
        vm.startPrank(organiser);
        choosingRam.selectRamIfNotSelected();
        vm.stopPrank();

        vm.warp(1728777600 + 1);
        vm.startPrank(player2);
        dussehra.killRavana();
        vm.stopPrank();
    }
```

We run the test using `forge test --match-test test_killRavanaAfter13thOfOctober -vvvv` and the result of this test will show a `pass`, indicating that the function call was successful despite being outside the designated period, thus confirming the `existence` of the vulnerability.

```
forge test --match-test test_killRavanaAfter13thOfOctober -vvvv
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[PASS] test_killRavanaAfter13thOfOctober() (gas: 271399)
Traces:
  [271399] CounterTest::test_killRavanaAfter13thOfOctober()
    ├─ [0] VM::startPrank(player1: [0x7026B763CBE7d4E72049EA67E89326432a50ef84])
    │   └─ ← [Return] 
    ├─ [0] VM::deal(player1: [0x7026B763CBE7d4E72049EA67E89326432a50ef84], 1000000000000000000 [1e18])
    │   └─ ← [Return] 
    ├─ [169497] Dussehra::enterPeopleWhoLikeRam{value: 1000000000000000000}()
    │   ├─ [94460] RamNFT::mintRamNFT(player1: [0x7026B763CBE7d4E72049EA67E89326432a50ef84])
    │   │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: player1: [0x7026B763CBE7d4E72049EA67E89326432a50ef84], tokenId: 0)
    │   │   └─ ← [Stop] 
    │   ├─ emit PeopleWhoLikeRamIsEntered(competitor: player1: [0x7026B763CBE7d4E72049EA67E89326432a50ef84])
    │   └─ ← [Stop] 
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    ├─ [0] VM::warp(1728691201 [1.728e9])
    │   └─ ← [Return] 
    ├─ [0] VM::startPrank(organiser: [0xe81f335f0c35d66819c4dF203d728f579880b4b1])
    │   └─ ← [Return] 
    ├─ [33984] ChoosingRam::selectRamIfNotSelected()
    │   ├─ [2360] RamNFT::organiser() [staticcall]
    │   │   └─ ← [Return] organiser: [0xe81f335f0c35d66819c4dF203d728f579880b4b1]
    │   ├─ [361] RamNFT::tokenCounter() [staticcall]
    │   │   └─ ← [Return] 1
    │   ├─ [1224] RamNFT::getCharacteristics(0) [staticcall]
    │   │   └─ ← [Return] CharacteristicsOfRam({ ram: 0x7026B763CBE7d4E72049EA67E89326432a50ef84, isJitaKrodhah: false, isDhyutimaan: false, isVidvaan: false, isAatmavan: false, isSatyavaakyah: false })
    │   └─ ← [Stop] 
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    ├─ [0] VM::warp(1728777601 [1.728e9])
    │   └─ ← [Return] 
    ├─ [0] VM::startPrank(player2: [0xEb0A3b7B96C1883858292F0039161abD287E3324])
    │   └─ ← [Return] 
    ├─ [37874] Dussehra::killRavana()
    │   ├─ [376] ChoosingRam::isRamSelected() [staticcall]
    │   │   └─ ← [Return] true
    │   ├─ [0] organiser::fallback{value: 500000000000000000}()
    │   │   └─ ← [Stop] 
    │   └─ ← [Stop] 
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    └─ ← [Stop] 

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.24ms (171.79µs CPU time)

Ran 1 test suite in 586.51ms (1.24ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
## Impact
This vulnerability can lead to the `killRavana` function being executed outside the intended time window, which can result in:

1. Premature or delayed execution of the function.
2. Financial discrepancies where the `organiser` might receive funds outside the designated period.
3. Compromised event integrity, as the timing of the key event phase (`killing Ravana`) would be inaccurate.

## Tools Used
1. Manual Code Review
2. Foundry

## Recommendations
To fix this vulnerability, the `killRavana` function should use accurate `timestamp` values that precisely represent the start and end times for `12th October 2024` and `13th October 2024`. Additionally, it's essential to verify the timestamps' correctness to prevent similar issues in the future.
```diff
function killRavana() public RamIsSelected {
-        if (block.timestamp < 1728691069) {
+       if (block.timestamp < 1728691200) {
            revert Dussehra__MahuratIsNotStart();
        }
-        if (block.timestamp > 1728777669) {
+       if (block.timestamp > 1728777600) {
            revert Dussehra__MahuratIsFinished();
        }
        IsRavanKilled = true;
        uint256 totalAmountByThePeople = WantToBeLikeRam.length * entranceFee;
        totalAmountGivenToRam = (totalAmountByThePeople * 50) / 100;
        (bool success, ) = organiser.call{value: totalAmountGivenToRam}("");
        require(success, "Failed to send money to organiser");
    }
```
Running the `PoC` again:
```
forge test --match-test test_killRavanaAfter13thOfOctober -vv
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[FAIL. Reason: Dussehra__MahuratIsFinished()] test_killRavanaAfter13thOfOctober() (gas: 236319)
Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 1.88ms (177.92µs CPU time)

Ran 1 test suite in 4.10ms (1.88ms CPU time): 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/Dussehra.t.sol:CounterTest
[FAIL. Reason: Dussehra__MahuratIsFinished()] test_killRavanaAfter13thOfOctober() (gas: 236319)

Encountered a total of 1 failing tests, 0 tests succeeded
```
After running the `PoC` again, we can verify that the vulnerability has been fixed.
## Conclusion

By correcting the `timestamp` values in the `killRavana` function, we have ensured that the function can only be executed within the intended time window. This change prevents any premature or delayed execution, preserving the event's integrity and ensuring the correct distribution of funds.