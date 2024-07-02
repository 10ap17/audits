# Incorrect Fee Calculation in getInputAmountBasedOnOutput Function            

### Relevant GitHub Links

https://github.com/Cyfrin/2024-06-t-swap/blob/d1783a0ae66f4f43f47cb045e51eca822cd059be/src/TSwapPool.sol#L291C9-L293C53

## Summary
The `getInputAmountBasedOnOutput` function in the `TSwap` contract incorrectly calculates the input amount required for a given output amount due to an error in the `fee` calculation. The function scales the amount by `10000` instead of `1000`, causing the protocol to take more tokens from `users` than intended. This results in `users` paying higher `fees` than expected.

## Vulnerability Details
The `getInputAmountBasedOnOutput` function is designed to calculate the amount of input tokens required to obtain a specified amount of output tokens. However, the function currently miscalculates the `fee`, scaling the amount by `10000` instead of the correct scale of `1000`. This error leads to the protocol deducting more tokens from `users` than necessary, resulting in excessive `fees` being charged. The incorrect calculation is as follows:

```solidity
return (inputReserves * outputAmount * 10000) / ((outputReserves - outputAmount) * 997);
```
The correct calculation should scale by 1,000 to accurately compute the fee:

```solidity
return (inputReserves * outputAmount * 1000) / ((outputReserves - outputAmount) * 997);
```
As a result, `users` swapping tokens via the `swapExactOutput` function will pay significantly more tokens than expected for their trades. This issue is exacerbated when `users` provide infinite allowance to the `TSwapPool` contract, as it exposes them to continuous overcharging.

## Impact
1. `Financial Loss to Users`: `Users` are charged higher `fees` than expected, leading to financial losses.
2. `User Trust`: `Users` can lose trust in the `TSwap`protocol due to unexpected and excessive `fees`.
3. `Potential Exploitation`: Malicious actors could exploit this flaw to trick `users` into unfavorable trades, ultimately draining liquidity from the pool.

## Tools Used
1. Manual Code Review

## Recommendations
To fix this vulnerability, the `fee` calculation in the `getInputAmountBasedOnOutput` function should be corrected to scale by `1000` instead of `10000`. The corrected code is as follows:

```diff
function getInputAmountBasedOnOutput(
    uint256 outputAmount,
    uint256 inputReserves,
    uint256 outputReserves
    ) 
    external 
    view 
    returns (uint256 inputAmount) 
{
-    return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
+    return (inputReserves * outputAmount * 1000) / ((outputReserves - outputAmount) * 997);
}
```
This change ensures that the fee calculation is accurate, preventing the protocol from overcharging users.