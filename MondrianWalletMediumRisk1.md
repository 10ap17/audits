# `_validateSignature` should return SIG_VALIDATION_FAILED on signature mismatch

## Severity
Medium Risk

## Relevant GitHub Links
https://github.com/Cyfrin/2024-05-Mondrian-Wallet/blob/main/contracts/MondrianWallet.sol

## Summary
The `_validateSignature` function should return `SIG_VALIDATION_FAILED` on a signature mismatch instead of always returning `SIG_VALIDATION_SUCCESS`. This ensures proper handling of invalid signatures and improves the contract's resilience to faulty transactions.

## Vulnerability Details
Currently, the `_validateSignature` function in the `Mondrian Wallet` contract is designed to always return `SIG_VALIDATION_SUCCESS`, regardless of whether the signature is valid or not. This approach does not correctly handle scenarios where the signature is invalid. The function should explicitly return `SIG_VALIDATION_FAILED` when a signature does not match the expected signer.

```
function _validateSignature(PackedUserOperation calldata userOp, bytes32 userOpHash)
        internal
        pure
        returns (uint256 validationData)
    {
        bytes32 hash = MessageHashUtils.toEthSignedMessageHash(userOpHash);
        ECDSA.recover(hash, userOp.signature);

        //always returning 0 for success
        return SIG_VALIDATION_SUCCESS;
    }
```
## Impact
Failure to properly handle signature validation errors will lead to the acceptance and processing of unauthorized transactions, undermining the security guarantees of the `Mondrian Wallet` contract. This can compromise the integrity and security of the wallet and will lead to unauthorized actions being executed.

## Tools Used
Manual code review

## Recommendations
Modify the `_validateSignature` function to return `SIG_VALIDATION_FAILED` when the signature does not match the expected signer. This provides a clear indication of signature validation failure and prevents the processing of unauthorized transactions.

```
function _validateSignature(PackedUserOperation calldata userOp, bytes32 userOpHash)
        internal
        view
        returns (uint256 validationData)
    {
        bytes32 hash = MessageHashUtils.toEthSignedMessageHash(userOpHash);
        address recoveredAddress = ECDSA.recover(hash, userOp.signature);

        // Check if the recovered signer matches the expected owner address
        if (recoveredAddress == owner()) {
            return SIG_VALIDATION_SUCCESS;
        } else {
            return SIG_VALIDATION_FAILED;
        }
    }
```
By making this change, the contract will correctly handle signature validation failures and improve its overall security by ensuring that only valid, authorized transactions are processed.
