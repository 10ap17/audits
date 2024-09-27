# Lack of Slippage Protection            

### Relevant GitHub Links

https://github.com/Cyfrin/2024-06-t-swap/blob/main/src/TSwapPool.sol

## Summary
The `swapExactOutput` function in the `TSwapPool` contract lacks `slippage` protection, exposing `users` to potential losses if the price changes significantly between the transaction initiation and completion. Similar to the `swapExactInput` function, `swapExactOutput` should include a parameter allowing users to specify the maximum amount of tokens they're willing to pay, protecting them from excessive slippage.

## Vulnerability Details
The `swapExactOutput` function facilitates swapping an exact amount of output tokens, calculating the required amount of input tokens based on current reserves. However, it does not allow users to set a `maximum amount` of input tokens they are willing to pay, leaving them vulnerable to `slippage`. In volatile markets, users might end up paying significantly more than anticipated, leading to potential losses.
The following is the current implementation of the `swapExactOutput` function:

```solidity
function swapExactOutput(
    IERC20 inputToken,
    IERC20 outputToken,
    uint256 outputAmount,
    uint64 deadline
)
    public
    revertIfZero(outputAmount)
    revertIfDeadlinePassed(deadline)
    returns (uint256 inputAmount)
{
    uint256 inputReserves = inputToken.balanceOf(address(this));
    uint256 outputReserves = outputToken.balanceOf(address(this));

    inputAmount = getInputAmountBasedOnOutput(outputAmount, inputReserves, outputReserves);

    _swap(
        inputToken,
        inputAmount,
        outputToken,
        outputAmount
    );
}
```

## Impact
1. `Slippage Risk`: `Users` will pay significantly more tokens than intended if the price changes rapidly, resulting in unexpected losses.
2. `User Trust`: Lack of `slippage` protection can erode user trust in the platform, as users may feel unprotected against market volatility.
3. `Protocol Vulnerability`: Without `slippage` protection, the protocol becomes less attractive to `users`, potentially reducing liquidity and trading volume.

## Tools Used
1. Manual Code Review

## Recommendations
To protect users from `slippage`, modify the `swapExactOutput` function to include a `maxInputAmount` parameter. This parameter allows `users` to specify the `maximum number` of input tokens they are willing to pay. If the calculated input amount exceeds this value, the transaction should revert.
```diff
function swapExactOutput(
    IERC20 inputToken,
+   uint256 maxInputAmount    
    IERC20 outputToken,
    uint256 outputAmount,
    uint64 deadline
)
    public
    revertIfZero(outputAmount)
    revertIfDeadlinePassed(deadline)
    returns (uint256 inputAmount)
{
    uint256 inputReserves = inputToken.balanceOf(address(this));
    uint256 outputReserves = outputToken.balanceOf(address(this));

    inputAmount = getInputAmountBasedOnOutput(outputAmount, inputReserves, outputReserves);

+   if (inputAmount > maxInputAmount) {
+       revert TSwapPool__OutputTooHigh(inputAmount, maxInputAmount);
+   }

    _swap(
        inputToken,
        inputAmount,
        outputToken,
        outputAmount
    );
}
```