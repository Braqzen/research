<!-- omit from toc -->
# Reth - Execution Client for Ethereum

An Ethereum node consists of two layers:

- An execution client (such as Reth)
- A consensus client (such as Lighthouse)

The execution client deals with transactions (validation, gossip), block execution and state transitions, and storage.

The consensus client deals with blocks (proposal, gossip, attesting to block validity), and tracking state and performance of validators.

**TABLE OF CONTENTS**

- [Overview](#overview)
  - [Initialization](#initialization)
    - [Config files](#config-files)
    - [Database](#database)
      - [Database Layer](#database-layer)
      - [Abstract Layer](#abstract-layer)
      - [Design Benefits](#design-benefits)
    - [Networking](#networking)
      - [Node Discovery](#node-discovery)
      - [Session Management](#session-management)
  - [Synchronization](#synchronization)
    - [Initial Sync](#initial-sync)
    - [Real-Time Sync](#real-time-sync)
      - [Transaction Management](#transaction-management)
        - [Validation, Mempool Inclusion and Transaction Gossip](#validation-mempool-inclusion-and-transaction-gossip)
        - [Block Creation](#block-creation)
      - [Block Management](#block-management)
- [Execution Extensions (ExEx)](#execution-extensions-exex)
  - [Notification System](#notification-system)
    - [Design Benefits](#design-benefits-1)
  - [Implementation](#implementation)
    - [ExEx](#exex)
    - [ExExManager](#exexmanager)
- [Crates](#crates)


## Overview

### Initialization

#### Config files

Reth's start point is ingress of config data which customizes the execution client.

There are many sections available for customization.

- `Stages`: Configures syncing of blockchain data, maintaining state, updating the database etc.
- `Peers`: Configures management of network connections such as limiting the number of peers, time between attempting reconnection and peer reputation scoring.
- `Sessions`: Configures individual network sessions between peers and handles request timeouts and buffer sizes per peer.
- `Prunning`: Configures data storage and enables specific segments to be pruned independently of the others. This includes receipts, storage and account histories, when to prune etc.

#### Database

Upon initialization Reth checks for the existence of its database, and if it's not found, a new database of predefined `tables` is created.

The database in Reth consists of two layers:

- [**Database Layer**](#database-layer): `MDBX`
- [**Abstract Layer**](#abstract-layer): Sits on top of the database

##### Database Layer

`MDBX` is an extremely fast, compact, and transactional key-value database that guarantees data integrity and performance through its adherence to ACID properties.

##### Abstract Layer

The abstract layer standardizes database interactions by defining an interface to interact with the database and a schema for storing data. The schema can be thought of as a collection of `tables` with `keys` and `values`, where both keys and values may be complex data structures that are encoded and decoded.

There are many `tables` because each `table` focuses on one type of data, such as `transactions`, `headers`, `receipts` etc.

##### Design Benefits

This design, coupled with the properties of `MDBX`, provides Reth with flexibility and high performance when handling data.

#### Networking

Reth utilizes `DevP2P` to create its routing table of peers and manage connections. The process consists of node discovery followed by establishing individual peer connections.

##### Node Discovery

The node discovery process consists of locating and maintaining active nodes within its routing table.

The protocol used is `Discv4`, built on `UDP`, which utilizes a distributed hash table (`DHT`), based on the Kademlia algorithm, to maintain peers based on their "distance" to the node. The distance is not geographical but instead derived between IDs and stored/grouped within "buckets".

Each node has its own `DHT` with a set number of buckets rather than each node containing a global list of all nodes.

1. Reth contains a list of known bootstrap nodes used to begin node discovery
2. Reth sends a `PING` message to a node and waits for a `PONG` message response
3. The `PONG` response is verified to come from the `PING` recipient
4. Post verification Reth sends a `FIND_NODE` message with an ID to populate its routing table of peers
5. The Kademlia algorithm is used to find the closest peers and a list of peers is sent back
6. Reth will update its routing table and repeat step 2 via the new peers until the table is populated
7. Periodically, Reth will `PING` its peers to prune/update its routing table

`Discv4` is currently in use until all execution clients support `Discv5`. `Discv5` may be enabled through the use of a flag and it will run simultaneously along `Discv4`.

##### Session Management

After peers have added each other to their routing tables a connection may be established.

1. Reth will initiate a `TCP` connection to its peer
2. `RLPx` is used to create a secure session for communication
3. A `HELLO` message is sent to negotiate which sub-protocols (and versions) may be used for communication
4. Peers select common protocols with their highest compatible versions
5. Each sub-protocol establishes a connection in its own way
   1. `Eth`: Status message (network id, blockhash, version...)
   2. `LES`: Status message (network id, blockhash, version...)
   3. `Whisper`: Handshake (version, capabilities...)
   4. `Swarm`: Handshake (version, capabilities...)

The sub-protocols are

1. `Eth`: Ethereum protocol used for tx/block propagation, blockchain and state synchronization
2. `Light Ethereum Sub-protocol (LES)`: Used by light clients to verify state
3. `Whisper`: P2P encrypted messaging used for privacy and anonymity
4. `Swarm`: Distributed storage / content sharing
5. `Snap`: TODO
6. `Witness (WIT)`: TODO

> NOTE: Reth may not support all of these and some of these may be deprecated.

### Synchronization

Synchronization in Reth can be split into two sections

- [Initial Sync](#initial-sync)
- [Real-Time Sync](#real-time-sync)

#### Initial Sync

When Reth comes online it must catch up to the head of the blockchain. It accomplishes this by requesting data from randomly selected peers.

The synchronization process utilizes a `pipeline` mechanism, which processes `stages` sequentially. If a `stage` is executed successfully, the subsequent `stage` is executed. Otherwise, any changes are unwound.

The `stages` include, but are not limited to:

- `Headers`: To avoid a "long-range attack", headers are requested in batches from the tip of the chain in descending order to the latest block in its database. Each header is validated and then stored.
- `Body`: Using the downloaded headers, Reth determines which block bodies to download. It pre-validate them by checking the ommers hash and transaction root against the block body, followed by adding each transaction from the block to its database.
- `SenderRecovery`: The transaction signer is recovered and stored for each transaction.
- `Execution`: The transactions from each block are executed, and state changes (such as updates to balances, bytecode, etc.) are applied.

#### Real-Time Sync 

After completing the initial synchronization with the blockchain, Reth performs two tasks to stay updated with the latest state of the network:

- [Transaction Management](#transaction-management)
- [Block Management](#block-management)

##### Transaction Management

###### Validation, Mempool Inclusion and Transaction Gossip

1. Reth exposes a `RPC` for users and peers to submit transactions
2. When a new transaction is received Reth performs basic validation such as checking the nonce, gas limit, signature verification etc. 
3. If the transaction passes validation, it is added to its mempool
4. The validated transaction is then broadcast to its peers

###### Block Creation

1. Reth also provides another `RPC` for consensus clients to retrieve neccessary data for block creation
2. When a consensus client requests transactions for a new block, Reth selects transactions from its mempool based on a specific ruleset, often prioritizing those with the highest gas price
3. Along with the selected transactions, Reth supplies additional data required for block creation, including the block header, state root, transaction root, and receipt root

##### Block Management

There are two scenarios in which Reth receives a block from the consensus client:

- Reth's block has reached consensus
- The network has produced a new block

In both cases the consensus client sends the block back to Reth for execution and storage.

1. Reth performs block validation to adhere to protocol rules
2. Reth prepares state for transaction execution in Revm
3. Revm executes transactions sequentially while applying state changes
4. Revm returns the outcome of its execution to Reth
5. Reth finalizes the state changes and updates its database

## Execution Extensions (ExEx)

Paradigm provides a definition of ExExs in their [blog](https://www.paradigm.xyz/2024/05/reth-exex) as:

> Post-execution hooks for building real-time, high performance and zero-operations off-chain infrastructure on top of Reth.

### Notification System

As Reth's state changes, notifications are emitted which the ExEx may use to derive its state and functionality.

Notification examples include:

- **Chain operations**: Notifications about whether a commit, revert or reorg has occured
- **Blocks**: Information about their transactions, receipts and state changes 

Upon processing an event the ExEx must send an event back to Reth to indicate that it has finished processing the event and it's safe to prune the associated data.

#### Design Benefits

The benefits of using ExExs include:

- Immediate processing of blockchain data
- Scaling infrastructure without altering the core functionality of Reth

### Implementation

#### ExEx

<!-- ExEx is a `Future` compiled into Reth, not a separate process -->
<!-- Eventually turn them into dynamically loaded plugins -->

#### ExExManager

<!-- ExExManager receives events from node and sends them to each ExEx, then back from ExEx to Reth -->

## Crates
