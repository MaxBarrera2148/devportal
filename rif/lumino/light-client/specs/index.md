---
layout: rsk
title: Light Client
tags: lumino, client, rif
description: "Lumino Light Client design and implementation specifications"
---

In its initial implementation, RIF Lumino users had to run a full node in order to open off-chain payment channels and make payments. 

Users and developers can now make payments from wallets and dapps, without the need to run a full node.

To achieve that, a Light Client is needed. A RIF Lumino Light Client is a piece of software which is able to:

- Support the full API of the RIF Lumino full node, that means being able to interact with the RIF Lumino network like if it were a full node.
- Run on the client side environment, i.e: browser, mobile/desktop apps, etc.

We identified two possible ways in which a Light Client could be implemented, detailed as follows.

### Option 1
Create different SDKs that can be run in client environments (ie: node.js SDK, Android SDK, iOS SDK). These SDKs implement the full RIF Lumino protocol, which means:
- They can interact with Lumino smart contracts.
- They implement the Lumino protocol specification. This basically means that they port the code of the full node to each client SDK.

*Advantages*

- There is no need for another node to connect to the network.
- You don't need to trust another node in order to make off-chain payments.

*Disadvantages*

- The Light Client will be "light" only because it will be able to run in a browser or mobile app, but the entire protocol and state machine of a full node must be implemented.
- We need to do this for every environment; we need to rewrite the code for each language or platform.

### Option 2
Create a RIF Lumino HUB and a Light Client library.

A *hub* will be a Full Node with extended capabilities:
1. Can work as a full node only if the user specifies it.
2. Allows Light Clients to connect to it.
3. Implements the Lumino protocol and is in charge of sending and receiving messages from/to light clients.
4. Interacts with the Light Client library to request the signing of the messages involved in the protocol.
5. Enforces secure communications between the Light Client and a the HUB.
6. Maintains connections at the transport layer for each Light Client registered.
7. Interacts with the Lumino smart contracts to open/close channels (on-chain interactions).

The *light client* will be a library that:
1. Provides the hub with the necessary signed messages (off-chain interactions).
2. Keeps a reduced state machine of its channels and partners, in order to validate the messages a node requests to be signed.
3. Can select a hub to trust and use.

*Advantages*

- We can reuse the already implemented protocol of the full node, making some adaptations and code restructuring.
- The Light Client library will be simpler, so its implementation in different languages will be easier and faster.

*Disadvantages*

- The Light Client library must trust the hub node to some extent.

### General Disadvantages for both options
- The clients must be online to receive and send payments.

## Selected alternative
After deep analysis, in which we take into consideration the following aspects:

- Faster adoption
- Security
- Easy implementation
- Software design correctness

We decided to follow the second approach.

## Specifications
As previously mentioned, for this approach we had to work on two different parts of the solution:

- **Node**: Modify the current node in order to accept and manage remote connections for Light Clients.
- **Clients SDK**: Implement the SDKs for the languages we want to support on the light client side. At the moment the only supported language is JavaScript (web and React native) but Support for Android and iOS is on the roadmap.

### Requirements
1. The nodes should be able to run under two different execution modes.
  - *Full mode*: the node fulfills the whole Lumino protocol, and can operate on its own.
  - *Hub mode*: the node is just an intermediary, working on behalf of the Light Clients connected to it.
2. Light clients must be able to register to the Lumino Hub. This means that they will communicate with the Lumino Hub and start an onboarding process.
3. A Hub must provide a way to accept Light Clients registrations. Light clients must provide at the onboarding:
    1. Address
    2. Password (matrix server name signed with the private key)
4. The Hub must receive the messages that are directed to light clients. This means that all the messages sent by other nodes to the light clients will be handled by the Hub.
5. When receiving a message, the Hub must have two different workflows:
    1. *Standard workflow*: the message destinatary is the Hub itself, so the node must work as usual.
    2. *Light Client workflow*: the Hub knows the lumino protocol, and will have to interact with the Light Client in order to sign the responses it must send to the rest of the network.
6. The communication between the Light Client and the Hub must be secure.
7. Each Hub maintains connections at the transport layer for each Light Client registered under its node.
8. The state machine of the Hub must be extended to handle state changes of the light clients. To put it simply: the node will have a collection of different state machines and after each message it will have to select which state machine should be affected.
9. The database of the Hub must store the information of channels and messages both for itself and the light clients registered to it.
10. All the operations must support RNS domain names.
11. The Hub will expose a REST API to the Light Client.
12. After the onboarding process the Light Client will receive an API-KEY that will have to be sent on each node interaction.

## Design specification for the Light Clients SDKs

### Component diagram

<div align="center"><img width="100%" src="/assets/img/lumino/component-diagram.png" alt=""/></div>

Our SDKs will interact with different components:

| Component | Description |
|-----------|-------------|
| Storage | The SDK will need to store data of different kinds in the environment were it will be executed (configuration, state data, etc.). This storage must be provided by the application that is including the light client (for a web app it can be local storage, for iOS it could be core data, etc.) |
| Web3 provider | One of the main reasons to have a light client is to allow the different kinds of clients to connect to RIF Lumino Networking without having a full node running, but keeping the private keys on the client side (and not in the node). Our SDKs will never come into contact with unencrypted private Keys, so the idea is that apps using the SDK have to provide a web3 provider that will be used to sign the different transactions needed by the light client. |
| RIF Notifier | A connection with a notifier is required in order to increase the security on the light client side, e.g. if a light client asks a hub to open a channel. In that case, it will have to validate that the node is behaving correctly and really opened the channel. The only way to do this is to listen on chain for particular events or transactions. To do this the light clients will be able to subscribe to notifiers which will inform them about events on the blockchain. |
| RIF Lumino Node | This is the most obvious connection, our light client will have to maintain a connection with a RIF Lumino HUB in order to interact with RIF Lumino Network. |  

&nbsp;

We have divided the internal SDK structure in different components.

| Component | Description |
|-----------|-------------|
| Light Client Service | This component will implement the actual logic of the light client. It will be the component in charge of calling and orchestrating the interactions with the rest of the components. |
| State machine service | This component handles the internal state machine of the light client. It will be in charge of: **A)** initializing a state machine **B)** performing the necessary state machine transitions based on received messages **C)** persisting the state machine to the storage and **D)** loading the state machine from the storage. |
| Signer | Component in charge of signing a transaction or a particular data stream. It will interact with the web3 provider. |
| Notifier Manager | Component in charge of interacting with the RIF Notifier. | 
| Storage Manager | This component will be the owner of the underlying storage, providing read and write access to it. |  

&nbsp;

## Light Client onboarding

The following diagram describes the process that takes place when a light client registers itself to a Lumino hub.

<div align="center"><img width="100%" src="/assets/img/lumino/lumino-client-onboarding.png" alt=""/></div>

### Open Channel

In order to open a channel the Light Client must call the endpoint `light_channels` using the `PUT` method. 

We could have made the light client send the open channel transaction directly to RSK without passing through the node, but for the sake of maintaining all the logic on the node and keeping the light client SDK as simple as possible, we decided to leave this logic on the node side.

It should send a JSON object in the body with the following structure:

```json
{
  "partner_address": "0x1234...",
  "creator_address": "0x2345...",
  "token_address": "0x3456...",
  "signed_tx": "0x4567...",
  "settle_timeout": 500
}
```

| Field | Description |
|-------|-------------|
| `partner_address` | With who we want to open a channel. |
| `creator_address` | Light client address. |
| `token_address` | Token of the channel. |
| `settle_timeout` | Amount of blocks to wait between close and settle channel. |
| `signed_tx` | Raw transaction to open the channel. |  

&nbsp;

### Deposit

In order to deposit token on a channel the Light Client must call the endpoint `light_channels` using the `PATCH` method. 

We could have made the light client send the open channel transaction directly to RSK without passing through the node, but for the sake of maintaining all the logic on the node and keeping the light client SDK as simple as possible, we decided to leave this logic on the node side.

It should send a JSON object in the body with the following structure:

```json
{
  "signed_approval_tx": "0x1234...",
  "signed_deposit_tx": "0x2345...",
  "total_deposit": 100000000000000000
}
```

| Field | Description |
|-------|-------------|
| `signed_approval_tx` | Raw transaction for the token approval tx. |
| `signed_deposit_tx` | Raw transaction for register the deposit in the token network. |
| `total_deposit` | Amount to deposit. | 

&nbsp;

### Interaction between node and light client during the payment process

The following diagram describes the logical message interactions required between the light client and a Lumino hub in order to make a payment.

<div align="center"><img id="lumino-interaction" width="100%" src="/assets/img/lumino/lumino-interaction.png" alt=""/></div>

Next you will see the diagram describing the interaction between the Light Client SDK and the Lumino Hub.

> Note: The following diagram is a draft version.

<div align="center"><img id="lumino-interaction-2" width="100%" src="/assets/img/lumino/lumino-interaction-2.png" alt=""/></div>

- The flow starts when a Light Client invokes the `initPayment` endpoint. This indicates to the node a new payment request made by the Light Client exists. This sets off a few actions made by the node:
    - It creates a new Payment entity with all the general data related to the payment.
    - It starts [the payment flow](#lumino-interaction)
        - This flow generates a few messages to sign by the light client (remember: the node never has access to the light client's private keys, so it has to ask the light client to sign the messages it needs to send to the payee)
- Every time the node needs some interaction with the light client it creates a new message in the `MessagesPending` table.
    - In order to get the messages in that table, the light client needs to long poll the `msg` endpoint using `GET` method.
    - When the light client retrieves a new message to process, it sends the message to the [Light Client Service component](#lumino-interaction-2). This service:
        - Checks if the message is valid: it should make sense based on its internal state.
        - If all the validations were fine, the service generates a new message response and sends it to the node.
        - After the Light Client Service processes the message and generates a  response to the node, it sends it to the `msg` endpoint using the `PUT` method.
