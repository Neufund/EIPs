## Preamble

    EIP: <to be assigned>
    Title: Token with snapshots
    Author: Marcin Rudolf <rudolfix@neufund.org>
    Type: Standard Track
    Category ERC
    Status: Discussion
    Created: 2017-10-26
    Requires ERC 20


## Simple Summary
An ERC20 interface extension to query past balances and total supply of a token.

## Abstract
The following proposal describes standard interface for querying token past states ( **token snapshots**). It also defines properties of ordered set (**snapshotting progression**) that is used to identify and thus query past states. Token state at upper bound of such progression is required be identical with token state as defined in ERC20 standard.

## Motivation
ERC20 tokens provides rights to own and transfer value by its holders. However many token applications require more complicated rights that are not transferable thus are impossible to implement with this standard. Examples of those rights are revenue disbursal (token holder should be able to claim past payouts), voting (vote cast should be final and cannot be transferred) and various rights to information, coupons and bonuses that are not voided when balance is transfered and can be claimed afterwards.
Standardized access to past token snapshots allows any contract, d-app and external party to implement such rights while keeping token contract itself simple and ERC20 compatible.

## Specification

### Snapshotting progression
Snapshotting progression defines valid snapshot ids against which token state can be queried. It is defined as ordered set of strictly increasing `uint256` integers greater than `0`. Snapshot id `0` always queries token state before any changes to total supply or balances (**initial state**). Snapshot ids need not be consecutive. New ids may be added at any moment in time. Last added id is called **current upper bound**.
Examples of valid progressions are block numbers, block timestamps or number of day boundaries from epoch.

### Token functions

#### currentSnapshotId

Returns current upper bound of snapshotting progression.
** NOTE ** token state with the id of current upper bound is mutable. Contracts should not rely on this state for any operations, in particular token cloning. Immutable state may always be queried with snapshot id of value `currentSnapshotId() - 1`.

``` js
function currentSnapshotId() public constant returns (uint256);
```

#### totalSupplyAt

Returns the total token supply for a snapshot with id `snapshotId`.

The function MUST `revert` if queried for a snapshot with id greater than current upper bound.
The function MUST return value of ERC20 [`totalSupply`](!https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md#totalsupply) function if queried for current upper bound.
The function MUST return value for all snapshot ids up until current upper bound even if total supply of the token didn't change.
The function SHOULD return initial token supply if queried for a snapshot id earlier than snapshot with lowest id stored by the token contract.

``` js
function totalSupplyAt(uint256 snapshotId) public constant returns(uint256);
```

### balanceOfAt

Returns the account balance of account with address `owner` for a snapshot with id `snapshotId`.

The function MUST `revert` if queried for a snapshot with id greater than current upper bound.
The function MUST return value of ERC20 [`balanceOf`](!https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md#balanceof) function if queried for current upper bound.
The function MUST return value for all snapshot ids up until current upper bound even if total supply of the token didn't change.
The function SHOULD return intial token balance of `owner` if queried for a snapshot id earlier than snapshot with lowest id stored by the token contract.

``` js
function balanceOfAt(address owner, uint256 snapshotId) public constant returns (uint256);
```

## Rationale
This proposal deliberately does not define any particular snapshotting progression as long as it conforms to specification above. Token implementations are free to choose current block number, day boundary from epoch, even event-triggered (like voting on snapshot id increase) or other progression depending on application of a token.

## References and Prior Art
* [MiniMe](!https://github.com/Giveth/minime) token implementation was first to provide 

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
