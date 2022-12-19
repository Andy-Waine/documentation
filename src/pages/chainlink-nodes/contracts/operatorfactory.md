---
layout: ../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Operator Factory"
---

The [factory](https://www.youtube.com/watch?v=Q1zZo4O_Ong) design pattern is a well know programming pattern: Rather than compiling and creating instances of a contract yourself, the _factory_ does it for you.
As explained in the [Operator guide](/chainlink-nodes/contracts/operator), `OperatorFactory` contract comes with these benefits:

- Node operators do not need to manually compile and deploy [Operator](/contracts/operator) or/and [Forwarder](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.7/AuthorizedForwarder.sol) contracts. They can deploy them directly from the factory. See [deploynewoperator](#deploynewoperator) , [deploynewforwarder](#deploynewforwarder) , and [deploynewoperatorandforwarder](#deploynewoperatorandforwarder) functions.
- Clients can verify if the factory deployed a given contract. See [created](#created) function.

## Api Reference

### Methods

##### typeAndVersion

```solidity
function typeAndVersion() external pure virtual returns (string)
```

The type and version of this contract.

###### Return Values

| Name | Type   | Description             |
| ---- | ------ | ----------------------- |
|      | string | Type and version string |

##### deployNewOperator

```solidity
function deployNewOperator() external returns (address)
```

Creates a new Operator contract with the msg.sender as owner. Emits `OperatorCreated` event.

##### deployNewOperatorAndForwarder

```solidity
function deployNewOperatorAndForwarder() external returns (address, address)
```

Creates a new Operator contract with the msg.sender as owner and a new Operator Forwarder with the Operator as the owner. Emits:

- [OperatorCreated](#operatorcreated) event.
- [AuthorizedForwarderCreated](#authorizedforwardercreated) event.

##### deployNewForwarder

```solidity
function deployNewForwarder() external returns (address)
```

Creates a new Forwarder contract with the msg.sender as owner. Emits [AuthorizedForwarderCreated](#authorizedforwardercreated) event.

##### deployNewForwarderAndTransferOwnership

```solidity
function deployNewForwarderAndTransferOwnership(address to, bytes message) external returns (address)
```

Creates a new Forwarder contract with the msg.sender as owner. Emits [AuthorizedForwarderCreated](#authorizedforwardercreated) event.

##### created

```solidity
function created(address query) external view returns (bool)
```

Indicates whether this factory deployed an address.

### Events

#### OperatorCreated

```solidity
event OperatorCreated(address operator, address owner, address sender)
```

#### AuthorizedForwarderCreated

```solidity
event AuthorizedForwarderCreated(address forwarder, address owner, address sender)
```