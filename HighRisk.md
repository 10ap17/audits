# `ECDSA.recover` should check against owner
## Severity
High Risk

## Relevant GitHub Links
GitHub Link

## Summary
The Mondrian Wallet contract's _validateSignature function lacks proper authentication checks, allowing potentially unauthorized users to execute transactions. This vulnerability arises from the absence of verification to ensure that the provided signature corresponds to the expected signer, the owner of the Mondrian Wallet.

## Vulnerability Details
The `_validateSignature` function in the `Mondrian Wallet` contract is responsible for verifying the authenticity of the signature included in a `UserOp`. However, it only checks the validity of the signature without verifying whether it was signed by the actual owner of the wallet. This oversight opens up the possibility of unauthorized users submitting transactions with valid, yet forged, signatures, leading to potential security breaches.
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
The vulnerability exposes the `Mondrian Wallet` contract to unauthorized transaction execution. Malicious actors could exploit this vulnerability to carry out unauthorized actions, compromising the integrity and security of the wallet. Additionally, if left unaddressed, this vulnerability will undermine user trust and confidence in the `Mondrian Wallet` platform, which will result in loss of users.

## Tools Used
Manual code review

## Recommendations
To mitigate this vulnerability, it is crucial to update the `_validateSignature` function to include proper authentication checks. This involves verifying that the provided signature corresponds to the expected signer, the owner of the wallet. This can be achieved by comparing the `recoveredAddress` address from the signature and the `owner` address. If the addresses match, the signature is considered valid, otherwise the signature is considered invalid. Additionally, the state mutability of the function has been changed from `pure` to `view`.

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