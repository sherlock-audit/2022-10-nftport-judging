GimelSec

medium

# Corruptible upgrade patterns in `ERC721NFTProduct.sol` and `ERC1155NFTProduct.sol`

## Summary

Both affected upgradeable contracts inherit a contract without gap storage slots, exposing them to the risk of potential storage overlap/corruption in future upgrade.

## Vulnerability Detail

A common problem in upgradeable/proxy contracts is mismanagement of storage layout, which boils down to overlap of different storage slots and potential type confusion.

The general way to avoid this is to inherit storage layout from a single “storage manager contract” as in Compound, and only append to the end of storage upon upgrade. Or define spare gap slots in each inherited contract to allow future upgrades to utilize those slots without having storage overlap with child contracts.

Both `ERC721NFTProduct` and `ERC1155NFTProduct` inherit `GranularRoles`, which does not have any gap slots prepared for potential upgrades. Thus exposing them to the risk of storage corruption upon future upgrades where additional variables need to be added to storage of `GranularRoles`

## Impact

Future upgrade may result in overlap of new `GranularRole` storage and storage of other contracts, corrupting the state of `ERC721NFTProduct` and `ERC1155NFTProduct` and potentially lead to loss of contract users.

## Code Snippet

https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/lib/GranularRoles.sol#L19
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC721NFTProduct.sol#L18
https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC1155NFTProduct.sol#L19


## Tool used

Manual Review

## Recommendation

Add spare slots in `GranularRole`:

```solidity
uint256[60] private __gap;
```

Here is spare slots example in [AccessControlUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/v4.7.3/contracts/access/AccessControlUpgradeable.sol#L259) which is inherited by `GranularRole`.
