<!-- omit from toc -->
# Revm - Ethereum Virtual Machine

<!-- Stack based, interpreted vm -->

**TABLE OF CONTENTS**

- [Overview](#overview)
- [Crates](#crates)
  - [Revm](#revm)
    - [Evm](#evm)
    - [Builder](#builder)
    - [Handler](#handler)
    - [Inspector](#inspector)
    - [State](#state)
    - [Journaled State](#journaled-state)
  - [Interpreter](#interpreter)
    - [Gas](#gas)
    - [Memory](#memory)
    - [Host](#host)
    - [Interpreter Action](#interpreter-action)
    - [Instruction Result](#instruction-result)
    - [Instructions](#instructions)
  - [Primitives](#primitives)
  - [Precompile](#precompile)

## Overview

## Crates

### Revm

<!-- Main crate that handles the call loop and host implementation, database handling, state journaling and powerful logic handlers that can be overwritten. This crate pulls Primitives, Interpreter and Precompiles together to deliver the rust evm. -->

#### Evm

<!-- `Evm` is the main struct, `EvmBuilder` creates and modifies the `Evm`, `Handler` modifies `Evm` logic. -->

<!-- Two parts:

- `Context` represent the state that is needed for execution
  - EvmContext is internal and contains Database, Environment, JournaledState and Precompiles
  - External context is fully generic (allow custom handlers to save state in runtime or allows hooks to be added)
- `Handler` contains list of functions that act as a logic -->

<!-- Runtime consists of list of functions from Handler that are called in predefined order. They are grouped by functionality on Verification, PreExecution, Execution, PostExecution and Instruction functions. -->

<!-- 2 loops:

- call loop
  - 1st loop - creates call frames, handles subcalls, it returns outputs and calls Interpreter loop to execute bytecode instructions
  - implements stack of frames
  - creates a `Frame` containing an `Interpreter` (per frame)
    - popped from stack, returns values that are pushed on top
    - subcalls which create new frames to put on top of the stack
  - empty stack == loop finishes
- interpreter loop
  - loops over bytecode opcodes and executes instructions based on the InstructionTable -->

<!-- The function of Evm is to start execution, but setting up what it is going to execute is done by EvmBuilder -->

#### Builder

<!-- split into stages with common functions that are renamed for clarity? -->

#### Handler

<!-- 5 handlers (used to manage tx execution process)
  - ValidationHandler
    - validate tx + block data
    - called prior to tx execution
  - PreExecutionHandler
    - functions called prior to execution such as loading access lists and precompiles
  - ExecutionHandler
    - tx execution and handles stack of call frames
  - PostExecutionHandler
    - finalizes execution and performs cleanup
    - ex. refunds gas, returns state changes + execution result, clears jouranl state
  - InstructionHandler

registers are functions that modify handler functions?
registers are called in the order they are registered -->

#### Inspector

<!-- `Inspector` - legacy interface for inspecting execution repurposed as a "handler register" -->

<!-- - used to execute and monitor tx, ex. gas usage, traces and prints custom messages during execution, modify execution?
  - called at various stages of execution -->

#### State

<!-- - fetches external state + storage
  - impl functionality on output of execution ex. caching while executing multiple tx's
  - has database abstractions which may be modified -->

#### Journaled State

<!-- `journaled_state` - journaling system to handle changes and reverts -->

<!-- - State management for accounts `Account`
  - tracks state changes in a journal `log` in case of a revert, and commits to db
  - `JournaledState` encapsulates the entire state of the blockchain, handles journaling of state changes `JournalEntry` and commits to db -->
  
### Interpreter

<!-- Event loop per `frame` that executes opcodes.

It handles: gas, contracts, memory, stack, returning execution results. -->

#### Gas

<!-- gas struct handling used gas, memory gas, refund gas, gas limit... -->

#### Memory

<!-- Each interpreter has its own local memory that it uses (while revm has a shared memory between all interpreters).

No memory limit but restrained by logarithmic growth of gas cost. -->

#### Host

<!-- interface for the interaction of the EVM interpreter with its environment
- account and storage access, creating logs, and invoking transactions.

This abstraction allows the EVM code to interact with the state of the Ethereum network in a generic way

Different implementations of the Host trait can be used to simulate different environments for testing or for connecting to different Ethereum-like networks. -->

#### Interpreter Action

<!-- collection of critical data structures used as internal models within the EVM -->

#### Instruction Result

<!-- definitions of the InstructionResult and SuccessOrHalt enum, which represent the possible outcomes of EVM instruction execution -->

<!-- The InstructionResult enum categorizes the different types of results that can arise from executing an EVM instruction.  -->

<!-- The SuccessOrHalt enum represents the outcome of a transaction execution, distinguishing successful operations, reversion, halting conditions, fatal external errors, and internal continuation. -->

#### Instructions

<!-- opcode enum
instruction struct
- contains the opcode
- list of bytes representing the operands for the instruction

The step function interprets an instruction. It uses the opcode to determine what operation to perform and then performs the operation using the operands in the instruction. -->

### Primitives

### Precompile

<!-- contains the implementation of the Ethereum precompile opcodes

Precompiles are a shortcut to execute a function implemented by the EVM itself, rather than an actual contract. Precompiled contracts are essentially predefined smart contracts on Ethereum, residing at hardcoded addresses and used for computationally heavy operations that are cheaper when implemented this way -->
