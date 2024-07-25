# Reth - Execution Client for Ethereum

To start an Ethereum node you need to run an execution client (such as Reth) and a consensus client (such as Lighthouse).

The execution layer deals with: tx validation, tx gossip, block execution, state transitions and storage.

The consensus layer deals with: block proposal, block gossip and attesting to block validity, tracking state and performance of validators.

## Reth Workflow Overview

### Initialization: Config files

Reth's starts point is ingress of config data which customizes the execution client.

There are many sections available for customization.

#### Stages

The stages section concerns itself with content such as, but not limited to, syncing of blockchain data, maintaining state and updating the database.

Reth utilizes a pipeline for processing stages. When one stage completes successfully the subsequent stage may begin.

#### Peers

The peers section concerns itself with management of network connections such as limiting the number of peers, time between attempting reconnection and peer reputation scoring.

#### Sessions

The session section concerns itself with individual network sessions between peers and handles request timeouts and buffer sizes per peer.


#### Pruning

The pruning section concerns itself with data storage and enables specific segments to be pruned independently of the others. This includes receipts, storage and account histories, when to prune etc.

### Initialization: Database

### Initialization: Networking



## Crates