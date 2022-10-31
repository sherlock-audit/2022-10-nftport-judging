0x0

high

# `updateRoles` Requires Administrative Role

## Summary

It's not possible to update/freeze non-admin roles with solely `UPDATE_CONTRACT_ROLE`.

## Vulnerability Detail

[`GranularRoles._updateRoles`](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/lib/GranularRoles.sol#L156)

[`ERC1155NFTProduct.update`](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC1155NFTProduct.sol#L223)

The docstring for `ERC1155NFTProduct.update` states that accounts with either roles `UPDATE_CONTRACT_ROLE` or `ADMIN_ROLE` should be able to update configuration items. The implementation does not reflect this as `GranularRoles._updateRoles` requires `msg.sender` to additionally have `ADMIN_ROLE`.

## Impact

- Updating the deployed template requires the caller to have both `UPDATE_CONTRACT_ROLE` and `ADMIN_ROLE` to be able to update contract configuration. This means the contract updater will have full control over administrative functions of the contract.

## Code Snippet

```solidity
function update(
    Config.Runtime calldata newConfig,
    RolesAddresses[] memory rolesAddresses,
    bool isRevokeNFTPortPermissions
) public onlyRole(UPDATE_CONTRACT_ROLE) {
```

```solidity
function _updateRoles(RolesAddresses[] memory rolesAddresses) internal {
    if (rolesAddresses.length > 0) {
        require(
            hasRole(ADMIN_ROLE, msg.sender),
            "GranularRoles: not an admin"
        );
```

## Tool used

Manual Review

## Recommendation

- Ensure that modifiers that validate if an account is able to access a function has the correct role
- Enforce a clear separation between roles `UPDATE_CONTRACT_ROLE` and `ADMIN_ROLE`
