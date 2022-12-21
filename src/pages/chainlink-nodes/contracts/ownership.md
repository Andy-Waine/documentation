---
layout: ../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Ownership"
---

[ConfirmedOwnerWithProposal](https://github.com/smartcontractkit/chainlink/edit/develop/contracts/src/v0.7/ConfirmedOwnerWithProposal.sol) is inherited by [Operator](/chainlink-nodes/contracts/operator) and [Forwarder](/chainlink-nodes/contracts/forwarder) contracts. It contains helpers for basic contract ownership.

## Api Reference

### Methods

#### transferOwnership

```solidity
function transferOwnership(address to) public
```

Allows an owner to begin transferring ownership to a new address.
Emit [OwnershipTransferRequested](#ownershiptransferrequested) event.

#### acceptOwnership

```solidity
function acceptOwnership() external
```

Allows an ownership transfer to be completed by the recipient.
Emit [OwnershipTransferred](#ownershiptransferred) event.

#### owner

```solidity
function owner() public view returns (address)
```

Get the current owner.

### Events

#### OwnershipTransferRequested

```solidity
event OwnershipTransferRequested(address from, address to)
```

#### OwnershipTransferred

```solidity
event OwnershipTransferred(address from, address to)
```