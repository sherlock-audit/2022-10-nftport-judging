0x0

medium

# Admin Cannot Update Token URI (721)

## Summary

`updateTokenUri` allows 721 individual token URIs to be updated from selected accounts, protected by a modifier. The docstring states that accounts with roles `UPDATE_TOKEN_ROLE` or `ADMIN_ROLE` may call this function. In the implementation this is not working for accounts with `ADMIN_ROLE`.

## Vulnerability Detail

[`ERC721NFTProduct.updateTokenUri `](https://github.com/sherlock-audit/2022-10-nftport/blob/main/evm-minting-master/contracts/templates/ERC721NFTProduct.sol#L124)

The modifier on this function allows accounts holding specific roles to perform the mint. Accounts holding `ADMIN_ROLE` may not call this without additionally being granted the `UPDATE_TOKEN_ROLE`.

## Impact

- Accounts holding `ADMIN_ROLE` will have to pay additional gas to modify the RBAC role to include the admin account

## Code Snippet

```solidity
function updateTokenUri(
    uint256 _tokenId,
    string memory _tokenUri,
    bool _isFreezeTokenUri
) public onlyRole(UPDATE_TOKEN_ROLE) {
```

## Tool used

Manual Review

## Recommendation

- Grant the role in the constructor at deploy time or update the modifier to validate if `msg.sender` holds roles `(UPDATE_TOKEN_ROLE || ADMIN_ROLE)`
