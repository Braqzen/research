# Reth - Execution Client for Ethereum

NOTE: WIP. Content may be incorrect.

To start an Ethereum node you need to run an execution client (such as Reth) and a consensus client (such as Lighthouse).

The execution layer deals with: tx validation, tx gossip, block execution, state transitions and storage.

The consensus layer deals with: block proposal, block gossip and attesting to block validity, tracking state and performance of validators.

## Reth Overview

### Initialization

#### Config files

Reth's starts point is ingress of config data which customizes the execution client.

There are many sections available for customization.

##### Stages

The stages section concerns itself with content such as, but not limited to, syncing of blockchain data, maintaining state and updating the database.

Reth utilizes a pipeline for processing stages. When one stage completes successfully the subsequent stage may begin.

##### Peers

The peers section concerns itself with management of network connections such as limiting the number of peers, time between attempting reconnection and peer reputation scoring.

##### Sessions

The session section concerns itself with individual network sessions between peers and handles request timeouts and buffer sizes per peer.

##### Pruning

The pruning section concerns itself with data storage and enables specific segments to be pruned independently of the others. This includes receipts, storage and account histories, when to prune etc.

#### Database

Upon initialization Reth checks for the existence of its database, and if it's not found, a new database of predefined `tables` is created.

The database in Reth consists of two layers:

- **Database Layer**: `MDBX`
- **Abstract Layer**: Sits on top of the database

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

##### Connecting to peers

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
5. `Snap`:
6. `Witness (WIT)`:

### Synchronization

Synchronization in Reth can be split into two parts

- Initial Sync
- Real-Time Sync

#### Initial Sync

When Reth comes online it must catch up to the head of the blockchain. It accomplishes this by requesting data from randomly selected peers.

The synchronization process utilizes a `pipeline` mechanism, which processes `stages` sequentially. If a `stage` is executed successfully, the subsequent `stage` is executed. Otherwise, any changes are unwound.

The `stages` include, but are not limited to:

- `Headers`: To avoid a "long-range attack", headers are requested in batches from the tip of the chain in descending order to the latest block in its database. Each header is validated and then stored.
- `Body`: Using the downloaded headers, Reth determines which block bodies to download. It pre-validate them by checking the ommers hash and transaction root against the block body, followed by adding each transaction from the block to its database.
- `SenderRecovery`: The transaction signer is recovered and stored for each transaction.
- `Execution`: The transactions from each block are executed, and state changes (such as updates to balances, bytecode, etc.) are applied.

#### Real-Time Sync 

## Crates
