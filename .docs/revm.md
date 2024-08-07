<!-- omit from toc -->
# Revm - Ethereum Virtual Machine

<!-- Stack based, interpreted vm -->

**TABLE OF CONTENTS**

- [Overview](#overview)
- [Crates](#crates)
  - [Revm](#revm)
  - [Interpreter](#interpreter)
  - [Primitives](#primitives)
  - [Precompile](#precompile)

## Overview

## Crates

### Revm

<!-- Main crate that handles the call loop and host implementation, database handling, state journaling and powerful logic handlers that can be overwritten. This crate pulls Primitives, Interpreter and Precompiles together to deliver the rust evm. -->

<!-- `Evm` is the main struct, `EvmBuilder` creates and modifies the `Evm`, `Handler` modifies `Evm` logic. -->

<!-- `Inspector` - legacy interface for inspecting execution repurposed as a "handler register" -->

<!-- `journaled_state` - journaling system to handle changes and reverts -->

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

### Interpreter

### Primitives

### Precompile
