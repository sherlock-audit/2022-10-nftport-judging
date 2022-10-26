GimelSec

medium

# Template implementations doesn't validate configurations properly

## Summary

In past audits, we have seen contract admins claim that invalidated configuration setters are fine since “admins are trustworthy”. However, cases such as [Nomad got drained for over $150M](https://twitter.com/samczsun/status/1554260106107179010) and [Misconfiguration in the Acala stablecoin project allows attacker to steal 1.2 billion aUSD](https://web3isgoinggreat.com/single/misconfiguration-in-the-acala-stablecoin-project-allows-attacker-to-steal-1-2-billion-ausd) have shown again and again that even trustable entities can make mistakes. Thus any fields that might potentially result in insolvency of protocol should be thoroughly checked.

NftPort template implementations often ignore checks for config fields. For the rest of the issue, we take `royalty` related fields as an example to illustrate potential consequences of misconfigurations. Notably, lack of check is not limited to `royalty`, but exists among most config fields.

Admins are allowed to set a wrong `royaltiesBps` which is higher than `ROYALTIES_BASIS`. `royaltyInfo()` will accept this invalid `royaltiesBps` and users will pay a large amount of royalty.

## Vulnerability Detail

EIP-2981 (NFT Royalty Standard) defines `royaltyInfo()` function that specifies how much to pay for a given sale price. In general, royalty should not be higher than 100%. [NFTCollection.sol](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/NFTCollection.sol#L348) checks that admins can't set royalties to more than 100%:
```solidity
    /// Validate a runtime configuration change
    function _validateRuntimeConfig(RuntimeConfig calldata config)
        internal
        view
    {
        // Can't set royalties to more than 100%
        require(config.royaltiesBps <= ROYALTIES_BASIS, "Royalties too high");

        ...
```

But `NFTCollection` only check `royaltiesBps` when admins call `updateConfig()`, it doesn't check `royaltiesBps` in `initialize()` function, leading to admins could set an invalid `royaltiesBps` (higher than 100%) when initializing contracts.

The same problem exists in ERC721NFTProduct and ERC1155NFTProduct. Both ERC721NFTProduct and ERC1155NFTProduct don't check `royaltiesBasisPoints` in `initialize()` function. Furthermore, these contracts also don't check `royaltiesBasisPoints` when admins call `update()` function. It means that admins could set an invalid `royaltiesBasisPoints` which may be higher than 100% in any time.

## Impact

EIP-2981 only defines `royaltyInfo()` that it should return royalty amount rather than royalty percentage. It means that if the contract has an invalid royalty percentage which is higher than 100%, `royaltyInfo()` doesn't revert and users will pay a large amount of royalty.

## Code Snippet

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/NFTCollection.sol#L348
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/NFTCollection.sol#L153
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC721NFTProduct.sol#L91
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC721NFTProduct.sol#L201
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC1155NFTProduct.sol#L96
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC1155NFTProduct.sol#L238


## Tool used

Manual Review

## Recommendation

Check `royaltiesBps <= ROYALTIES_BASIS` both in `initialize()` and `update()` functions.
