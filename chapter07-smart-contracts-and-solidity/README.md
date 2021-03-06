# Chapter 07. Smart Contracts and Solidity

- EOAs are controlled by users, often via software such as a wallet application that is external to
  the Ethereum platform
- Contract accounts are controlled by program code (also commonly referred to as 
  "smart contracts") that is executed by the Ethereum Virtual Machine

## What Is a Smart Contract?

- **WHAT**: Immutable computer programs that run deterministically in the context of an Ethereum
  Virtual Machine as part of the Ethereum network protocol, i.e., on the decentralized Ethereum
  world computer
  - **Computer programs**: Smart contracts are simply computer program
  - **Immutable**: Once deployed, the code of a smart contract cannot change
  - **Deterministic**: Always the same output given the same tx and blockchain state
  - **EVM context**: Smart contracts can access their own state, the context of the transaction
    that called them, and some information about the most recent blocks
  - **Decentralized world computer**: All instances of the EVM operate on the same initial state
    and produce the same final state, the system as a whole operates as a single "world computer"

## Life Cycle of a Smart Contract

- Life cycle
  - Written in a high-level language, such as Solidity
  - Compiled to the low-level bytecode that runs in the EVM
  - Deployed on the Ethereum platform using a special contract creation transaction
- Each contract is identified by an Ethereum address, which is derived from the contract creation
  transaction as a function of the originating account and nonce
- A contract address as a recipient is for receiving funds or calling one of the contract's
  functions
- **Contracts only run if they are called by a transaction**
- Transactions are **atomic**
  - On success, any changes in the global state (contracts, accounts, etc.) is recorded only
  - On failure, all effects are "rolled back"/reverted
- Contract deletion
  - A contract can be deleted with the EVM opcode called `SELFDESTRUCT`
  - Any txs sent to the deleted contract account address will result in no code execution
  - `SELFDESTRUCT` costs "negative gas," a gas refund, thereby incentivizing the release of
    network client resources from the deletion of stored state
  - The `SELFDESTRUCT` capability will only be available if the contract author programmed the
    smart contract to have that functionality

## Introduction to Ethereum High-Level Languages

- The EVM is a virtual machine that runs a special form of code called **EVM bytecode**
- **WHY NOT BYTECODE**
  - EVM bytecodes is rather unwieldy and very difficult for programmers to read and understand
  - Programs writting in a high-level language can be compiled down to bytecodes
- Two broad programming paradigms

  - Declarative (a.k.a, functional), including Haskell and SQL
    - Makes it easier to understand how a program will behave: since it has no side effects, any
      part of a program can be understood in isolation
  - Imperative (a.k.a. procedural), including C++ and Java

    - More commonly used by programmers
    - It can be very difficult to write programs that execute exactly as expected
    - The ability of any part of the program to change the state of any other makes it difficult
      to reason about a program's execution and introduces many opportunities for bugs

    > The hybrid ones include Lisp, JavaScript and Python

- In smart contracts, bugs literally cost money
  - Declarative languages are recommended
  - But imperative ones are favored by programmers
- Supported high-level languages are

  | Language | Style       | Description                                                                                                                                                                                 |
  | -------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | LLL      | Declarative | Lisp-like syntax                                                                                                                                                                            |
  | Serpent  | Imperative  | Python-like syntax. Can also be used to write functional (declarative) code, though it is not entirely free of side effects                                                                 |
  | Solidity | Imperative  | JS/C++/Java-like syntax. **The most popular and frequently used language for Ethereum smart contracts**                                                                                     |
  | Vyper    | Declarative | Similar to Serpent and again with Python-like syntax. Intended to get closer to a pure-functional Python-like language than Serpent, but not to replace Serpent                             |
  | Bamboo   | N/A         | Influenced by Erlang, with explicit state transitions and without iterative flows (loops). Intended to reduce side effects and increase auditability. Very new and yet to be widely adopted |

## Building a Smart Contract with Solidity

- Solidity is now developed and maintained as an independent project 
  ([ethereum/solidity](https://github.com/ethereum/solidity)) on Github
- The main "product" of the Solidity project is the Solidity compiler, `solc`, which converts
  programs written in the Solidity language to EVM bytecode
- The project also manages the important **application binary interface** (ABI) standard for
  Ethereum smart contracts
- Each version of the Solidity compiler corresponds to and compiles a specific version of the
  Solidity language

### Selecting a Version of Solidity

- Solidity follows [semantic versioning](https://semver.org/), which specifies version numbers
  structured as three numbers separated by dots: `MAJOR.MINOR.PATCH`

  - The "major" number is incremented for major and backward-incompatible changes
  - The "minor" number is incremented as backward-compatible features are added in between major
    releases
  - The "patch" number is incremented for backward-compatible bug fixes

  > In practice, Solidity treats the "minor" number as if it were the major version and the
  > "patch" number as if it were the minor version.

- Solidity programs can contain a pragma directive that specifies the minimum and maximum versions
  of Solidity that it is compatible with, and can be used to compile your contract

### Download and Install

- Detailed in [the Solidity documentation](https://solidity.readthedocs.io/en/latest/installing-solidity.html)
- In my case, I would use the docker-based version encapsulated as the [solc.sh](../solc.sh) script
  - Contracts to compile MUST be placed in the `contracts` directory

### Development Environment

- Desktop-based text editors
  - Emacs
  - Vim
  - Atom
- Web-based text editors
  - [Remix IDE](https://remix.ethereum.org/#optimize=false&version=soljson-v0.5.1+commit.c8a2cb62.js)
  - [EthFiddle](https://ethfiddle.com/)
- Simply save your program source code with a `.sol` extension and it will be recognized by the
  solidity compiler as a Solidity program

### Writing a Simple Solidity Program

- As [Faucet01.sol](examples/contracts/Faucet01.sol)

### Compiling with the Solidity Compiler (solc)

- Use the `--bin` and `--optimize` arguments of solc to produce an optimized binary of our example
  contract

  ```bash
  cd examples

  docker run --rm -v ${PWD}/contracts:/contracts --workdir /contracts ethereum/solc:0.7.0 --bin --optimize Faucet01.sol

  # Output
  ======= Faucet01.sol:Faucet =======
  Binary:
  608060405234801561001057600080fd5b5060cd8061001f6000396000f3fe608060405260043610601f5760003560e01c80632e1a7d4d14602a576025565b36602557005b600080fd5b348015603557600080fd5b50605060048036036020811015604a57600080fd5b50356052565b005b68056bc75e2d63100000811115606757600080fd5b604051339082156108fc029083906000818181858888f193505050501580156093573d6000803e3d6000fd5b505056fea264697066735822122038a17e21e07b9341fad71fcf3b53f21d72441ad65e764de5c92191b57804681964736f6c63430007000033
  ```

## The Ethereum Contract ABI

- In computer software, an **application binary interface** is an interface between two program
  modules; often, between the **operating system** and **user programs**
- An ABI defines how data structures and functions are accessed in **machine code**
- In Ethereum, the ABI is used to encode contract calls for the EVM and to read data out of
  transactions
- The purpose of an ABI is to
  - Define the functions in the contract that can be invoked
  - Describe how each function will accept arguments and return its result
- A contract's ABI is specified as a JSON array of
  - Function descriptions as `(type, name, inputs, outputs, constant, payable)`
  - Events as `(type, name, inputs, anonymous)`
- ABI produced by `solc` with `--abi` option
  ```bash
  cd examples

  docker run --rm -v ${PWD}/contracts:/contracts --workdir /contracts ethereum/solc:0.7.0 --abi Faucet01.sol 

  # Output
  ======= Faucet01.sol:Faucet =======
  Contract JSON ABI
  [{"inputs":[{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"withdraw","outputs":[],"stateMutability":"nonpayable","type":"function"},{"stateMutability":"payable","type":"receive"}]
  ```

- Interaction with a contract requires only
  - An ABI
  - The address where the contract has been deployed

### Selecting a Solidity Compiler and Language Version

- **Problem**: A contract written in a specific version of Solidity is given to different version
  of Solidity compilers
- **Solution**: Solidity offers a `compiler directive` known as a `version pragma` that instructs
  the compiler that the program expects a specific compiler (and language) version
- **Pragma directives are not compiled into EVM bytecode**

  - They are only used by the compiler to check compatibility
  - If missing, a warning will be reported

    - Tested with [Faucet2.sol](examples/contracts/Faucet02.sol) commenting out the `pragma` directive

      ```bash
      docker run --rm -v ${PWD}/contracts:/contracts --workdir /contracts ethereum/solc:0.7.0 --abi Faucet02.sol 

      # Output
      Warning: Source file does not specify required compiler version! Consider adding "pragma solidity ^0.7.0;"
      --> Faucet02.sol
      ...
      ```

- Adding a version pragma is a best practice, as it avoids problems with mismatched compiler and
  language versions

## Programming with Solidity

### Data Types

|                           Type | Description                                                                                                                                                                   |
| -----------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|               Boolean (`bool`) | `true`/`false`, with logical operators `!`(not), `&&`(add), `\|\|`(or), `==`(equal), `!=`(not equal)                                                                          |
|         Integer (`int`,`uint`) | Signed (int) and unsigned (uint) integers, declared in increments of 8 bits from `int8` to `uint256`. `int`/`uint` aliases `int256`/`uint256`                                 |
| Fixed point (`fixed`/`ufixed`) | Fixed-point numbers, declared with `(u)fixedMxN` where `M` is the size in bits (increments of `8` up to `256`) and `N` is the number of decimals after the point (up to `18`) |
|            Address (`address`) | A 20-byte Ethereum address with many helpful member functions, the main ones being `balance` (returns the account balance) and `transfer`                                     |
|           Byte array (_fixed_) | Fixed-size arrays of bytes, declared with `bytes1` up to `bytes32`                                                                                                            |
|         Byte array (_dynamic_) | Variable-sized arrays of bytes, declared with `bytes` or `string`                                                                                                             |
|                           Enum | User-defined type for enumerating discrete values: `enum NAME {LABEL1, LABEL2, ...}`                                                                                          |
|                         Arrays | An array of any type, either fixed or dynamic: `uint32[][5]` is a fixed-size array of five dynamic arrays of unsigned integers                                                |
|                         Struct | User-defined data containers for grouping variables: `struct NAME {TYPE1 VARIABLE1; TYPE2 VARIABLE2; ...}`                                                                    |
|                        Mapping | Hash lookup tables for `key => value` pairs: `mapping(KEY_TYPE => VALUE_TYPE) NAME`                                                                                           |

And various value literals as

|     Literal | Description                                                                                                                     |
| ----------: | :------------------------------------------------------------------------------------------------------------------------------ |
|  Time units | The units `seconds`, `minutes`, `hours`, and `days` can be used as suffixes, converting to multiples of the base unit `seconds` |
| Ether units | The units `wei`, `szabo`, `finney`, and `ether` can be used as suffixes, converting to multiples of the base unit `wei`         |

> Improve our code by using the unit multiplier ether, to express the value in `ether` instead of
> `wei`

### Predefined Global Variables and Functions

- Variables as
  - `block`
  - `msg`
  - `tx`
- Functions exposed as EVM opcodes

#### Transaction/message call context

- The `msg` object is the transaction call (EOA originated) or message call (contract originated)
  that launched this contract execution
- It contains a number of useful attributes

  | Attribute | Description                                                                                                                                                      |
  | --------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  |  `sender` | It represents the `address` that initiated this contract call, not necessarily the originating EOA that sent the transaction (contract address is also possible) |
  |   `value` | The value of ether sent with this call (in `wei`)                                                                                                                |
  |     `gas` | The amount of gas left in the gas supply of this execution environment. This was deprecated in Solidity **v0.4.21** and replaced by the `gasleft` function       |
  |    `data` | The data payload of this call into our contract                                                                                                                  |
  |     `sig` | The first four bytes of the data payload, which is the function selector                                                                                         |

> Whenever a contract calls another contract, the values of all the attributes of `msg` change to
> reflect the new caller's information. The only exception to this is the `delegatecall` function

#### Transaction context

- Expressed as the `tx` object with information

  |  Attribute | Description                                                                   |
  | ---------: | :---------------------------------------------------------------------------- |
  | `gasprice` | The gas price in the calling transaction                                      |
  |   `origin` | The address of the originating EOA for this transaction. **WARNING: unsafe!** |

#### Block context

- Expressed as the `block` object containing information

  |    Attribute | Description                                                                                       |
  | -----------: | :------------------------------------------------------------------------------------------------ |
  |   `coinbase` | The **address** of the recipient of the current block's fees and block reward                     |
  | `difficulty` | The difficulty (proof of work) of the current block                                               |
  |   `gaslimit` | The maximum amount of gas that can be spent across all transactions included in the current block |
  |     `number` | The current block number (blockchain height)                                                      |
  |  `timestamp` | The timestamp placed in the current block by the miner (number of seconds since the Unix epoch)   |

> The block hash of the specified block number, up to 256 blocks in the past can be queried by the global `blockhash(blockNumber)`

#### `address` object

|          Attribute | Description                                                                                                                                                                     |
| -----------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|          `balance` | The balance of the address, in `wei`. For example, the current contract balance is `address(this).balance`                                                                      |
| `transfer(amount)` | Transfers the amount (in `wei`) to this address, throwing an exception on any error                                                                                             |
|     `send(amount)` | Similar to `transfer`, only instead of throwing an exception, it returns `false` on error. **WARNING: always check the return value of send**                                   |
|    `call(payload)` | Low-level `CALL` function -- can construct an arbitrary message call with a data payload. Returns `false` on error. **WARNING: unsafe**                                         |
|   `delegatecall()` | Low-level `DELEGATECALL` function, like `callcode(...)` but with the full `msg` context seen by the current contract. Returns `false` on error. **WARNING: advanced use only!** |

> As of v0.5.0, function `callcode` is now disallowed (in favor of delegatecall). It is still possible to use it via inline assembly.

#### Built-in functions

|                                   Function | Description                                                                                       |
| -----------------------------------------: | :------------------------------------------------------------------------------------------------ |
|                          `addmod`,`mulmod` | For modulo addition and multiplication                                                            |
| `keccak256`, `sha256`, `sha3`, `ripemd160` | Functions to calculate hashes with various standard hash algorithms                               |
|                                `ecrecover` | Recovers the address used to sign a message from the signature                                    |
|          `selfdestruct(recipient_address)` | Deletes the current contract, sending any remaining ether in the account to the recipient address |
|                                     `this` | The address of the currently executing contract account                                           |

### Contract Definition

- Signaled by the `contract` keyword
- 2 other similar objects are
  - `interface` is structured exactly like a `contract`, except none of the functions are defined,
    they are only declared
    > This type of declaration is often called a `stub`
  - `library` is meant to be deployed only once and used by other contracts, using the
    `delegatecall` method

### Functions

- Syntax of declaration goes as

  ```solidity
  function FunctionName([parameters]) {public|private|internal|external} [pure|view|payable] [modifiers] [returns (return types)]
  ```

  - `FunctionName`
    - Defines the name of the function, which is used to call the function in a transaction (from
      an EOA), from another contract, or even from within the same contract.
    - Especially, the so-called `fallback` function is defined without a name, which is called
      when no other function is called. The `fallback` function cannot have any arguments or
      return anything
  - `parameters`
    - The arguments that must be passed to the function, with their **names** and **types**
  - `public`/`private`/`internal`/`external` specify the function's visibility
    - `public` is the default; such functions can be called by other contracts or EOA
      transactions, or from within the contract
    - `external` is like `public`, except the decorated functions cannot be called from within the
      contract unless explicitly prefixed with the keyword `this`
    - `internal` make functions only accessible from within the contract -- they cannot be called
      by another contract or EOA transaction. They can be called by derived contracts (those that
      inherit this one)
    - `private` is like `internal`, except the decorated functions cannot be called by derived
      contracts
  - `pure`/`view`/`payable` affects behaviors of the functions
    - `view` marks a function promising not to modify any state
    - `pure` marks functions neither reading nor writing any variables in storage
    - `payable` marks the only functions for accepting incoming payments
      > (TODO: EXPLAIN WHY) 2 exceptions are coinbase payments and `SELFDESTRUCT` inheritance will
      > be paid even if the fallback function is not declared as `payable`

> As of v0.5.0, `constant` aliasing `view` is disallowed

### Contract Constructor and `selfdestruct`

- When a contract is created, it also runs the `constructor` function if one exists, to initialize
  the state of the contract
- The constructor function is optional
- Constructors must be defined using the `constructor` keyword as

  ```solidity
  pragma ^0.4.22

  contract MEContract {
    constructor () {
      // This is the constructor
    }
  }
  ```

- To summarize, a contract's life cycle
  - Starts with a creation transaction from an EOA or contract account
  - If there is a constructor, it is executed as part of contract creation, to initialize the
    state of the contract as it is being created, and is then discarded.
  - Finally, contract can be destructed
- Contracts are destroyed by a special EVM opcode called `SELFDESTRUCT` exposed as a high-level
  built-in function as

  ```solidity
  // recipient is the address to receive any remaining ether balance
  selfdestruct(address recipient)
  ```

- Only the explicitly declared `selfdestruct` command can delete a contract

### Adding a `Constructor` and `selfdestruct` to Our Faucet Example

- Use case: Record the EOA as the creator of contract as `owner` in constructor, and enforce that
  only `owner` can invoke `selfdestruct`
- Demo goes as follows
  1. Open a quickstart workspace in Ganache
  2. Deploy contract [Faucet][Faucet03] to Ganache
      ```bash
      cd examples/constructor-and-selfdestruct

      # the key is picked from Ganache
      go run deploy.go -k 80921f75685414f80062edec828b9cd86bc1efbd568c6ea4c06da1a5098e89a4

      # Output
      gasPrice = 20000000000
       account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
        txHash = 0xf200658a9936a09cb3a9ab8b9385d008ac2170a1583a14bd916737f7c65e9de9
      ```

      The corresponding tx in Ganache goes as 

      ![](images/constructor-and-selfdestruct/deploy-tx.png)

  3. Delete the contract by calling `destroy()` of the contract

     - By nonowner would trigger error
        ```bash
        cd examples/constructor-and-selfdestruct

        # -k: non-owner key picked from Ganache
        # -tx: tx hash generated in the previous step
        go run destruct.go -k 6540cc8e3afc60da9615f8ec0338194dc2288712efb1a6944a00a5c5f8d8ec1b --tx 0xf200658a9936a09cb3a9ab8b9385d008ac2170a1583a14bd916737f7c65e9de9

        # Output
        panic: VM Exception while processing transaction: revert

        ...
        ```
     - By owner is fine
        ```bash
        cd examples/constructor-and-selfdestruct

        # -k: owner key picked from Ganache
        # -tx: tx hash generated in the previous step
        go run destruct.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --tx 0xf200658a9936a09cb3a9ab8b9385d008ac2170a1583a14bd916737f7c65e9de9 --nonce 1

        # Output
        gasPrice = 20000000000
         account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
          txHash = 0x6ee1d726c74ce9a4a363329c9c1da75cf97b637071868ba169b7b69e2e59fe0c
        ```

        Tx in Ganache looks like 
        ![destruct tx ok](./images/constructor-and-selfdestruct/destruct-tx-ok.png)

  4. After successful deletion, fetch the storage of the contract should produce `0x0` meaning
    contract has been deleted

      ```bash
      cd examples/constructor-and-selfdestruct

      go run ping.go --tx 0xf200658a9936a09cb3a9ab8b9385d008ac2170a1583a14bd916737f7c65e9de9

      # Output
      result: 0x0
      ```

[Faucet03]: examples/constructor-and-selfdestruct/contracts/Faucet03.sol

  > The run-once-only constructor renders the `owner` field constant once set

### Function Modifiers

- Modifiers are most often used to create conditions that apply to many functions within a contract
- An access control pattern goes as (requiring the `msg.sender` to be `owner`)

  ```
  modifier onlyOwner {
    require(msg.sender == owner);
    _;
  }
  ```

- The modifier is "wrapped around" the modified function, placing its code in the location
  identified by the underscore `_` placeholder
- More than one modifier can be applied to a function; they are applied in the sequence they are
  declared, as a comma-separated list
- They are most often used for access control, but they are quite versatile and can be used for a
  variety of other purposes
- Inside a modifier, you can access all the values (variables and arguments) visible to the
  modified function, but not vice verse
- The code sample goes as [Faucet04.sol](examples/contracts/Faucet04.sol)

### Contract Inheritance

- **WHAT**: A mechanism for extending a base contract with additional functionality
- **HOW**: Specify a parent contract with the keyword `is`

  ```solidity
  contract Child is Parent {
    ...
  }
  ```

- The `Child` contract inherits all the methods, functionality, and variables of Parent
- Multiple inheritance is specified by comma-separated contract names after the keyword `is`

  ```solidity
  contract Child is Parent1, Parent2 {
    ...
  }
  ```

- **WHY**: Write our contracts to achieve modularity, extensibility, and reuse
- Sample code goes as [Faucet05.sol](examples/contracts/Faucet05.sol)
  - A `owned` contract with the constructor and destructor, together with access control for an
    owner, assigned on construction
  - `Faucet` contract rebased on `mortal` which is inherited from `owned`

### Error Handling (assert, require, revert)

- **Atomicity** of contract execution: When a contract terminates with an error, all the state
  changes (changes to variables, balances, etc.) are reverted, all the way up the chain of
  contract calls if more than one contract was called
- 3 keywords (as of v0.5.6)

|                 Keyword | Description                                                                                                           |
| ----------------------: | :-------------------------------------------------------------------------------------------------------------------- |
|          `assert(bool)` | Should only be used to test for internal errors, and to check invariants                                              |
| `require(bool [, msg])` | Used to test inputs (such as function arguments or transaction fields), setting our expectations for those conditions |
|           `revert(msg)` | Used to flag an error and revert the current call                                                                     |

- Certain conditions in a contract will generate errors regardless of explicit check
- It might be better to check explicitly and provide a clear error message on the system-generated
  errors

- Code sample goes as [Faucet06.sol](examples/contracts/Faucet06.sol)

### Events

- The tx receipt contains _log_ entries providing information about the actions that occurred
  during the execution of the transaction
- Events are the Solidity high-level objects that are used to construct these logs
- Events are especially useful for light clients and DApp services, which can "watch" for specific
  events and report them to the user interface, or make a change in the state of the application
  to reflect an event in an underlying contract.
- Event objects take arguments that are serialized and recorded in the transaction logs, in the
  blockchain
- Supplying the keyword `indexed` before an argument of type `event` makes the value part of an
  indexed table (hash table) that can be searched or filtered by an application
- Event are triggered with the `emit` keyword

#### Catching events

- Events are a very useful mechanism, not only for intra-contract communication, but also for
  debugging during development
- Demo goes as follows
  1. Start a Ganache network
  2. Deploy [Faucet][examples/events/contracts/Faucet08.sol] to Ganache
     ```bash
     cd exmaples/events

     # key is fetch from the Ganache account list
     go run deploy.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 4

     # Output
     gasPrice = 20000000000
      account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
       txHash = 0x61555903d0a6911ae0917eecee46fd0b343ed15cfdb72caba96983ea2a718f55
     ```
  3. Once the tx is confirmed, deposit some amount into the contract to trigger the event in the
    fallback functions by [deposit.go](examples/events/deposit.go)

      ```bash
      cd exmaples/events

      go run deposit.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 5 --tx 0x61555903d0a6911ae0917eecee46fd0b343ed15cfdb72caba96983ea2a718f55

      # Output
      gasPrice = 20000000000
      account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
        txHash = 0xde0cf822f4521ffe82d8b9c16ff4c2e4a638cd2324ea1842a4f0dbc409f8b235
      ```
  4. Check the `Deposit` event as [deposit_logging.go](examples/events/deposit_logging.go)

      ```bash
      cd exmaples/events

      go run deposit_logging.go --tx 0xde0cf822f4521ffe82d8b9c16ff4c2e4a638cd2324ea1842a4f0dbc409f8b235 -a 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
      ```

      No output means good.

  5. Withdraw some amount out of the contract to trigger the `Withdrawal` event in the `withdraw()`
    as [withdraw.go](examples/events/withdraw.go)
      ```bash
      cd exmaples/events

      go run withdraw.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 9 --tx 0x61555903d0a6911ae0917eecee46fd0b343ed15cfdb72caba96983ea2a718f55

      # Output
      gasPrice = 20000000000
       account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
        txHash = 0xf01d63493822311093c8f5472bd5f6f210f521d62cad378f7aa88ab01c4ed57c
      ```
  6. Check the `Withdrawal` event as [withdrawal_logging.go](examples/events/withdrawal_logging.go),
    where no output means good

      ```bash
      cd exmaples/events

      go run withdrawal_logging.go --tx 0xf01d63493822311093c8f5472bd5f6f210f521d62cad378f7aa88ab01c4ed57c -a 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
      ```

### Calling Other Contracts (`send`, `call`, `callcode`, `delegatecall`)

- Calling other contracts from within your contract is a very useful but potentially dangerous
  operation
- The risks arise from the fact that you may not know much about a contract you are calling into
  or that is calling into your contract

#### Creating a new instance

- The safest way to call another contract is if you create that other contract yourself
- Contract instance can be created with initial ether by means of `value(amount)` function

- Demo

  ```solidity
  // import "Faucet.sol" // import the contract if it resides in other files

  contract Token is mortal {
    Faucet _faucet;

    constructor() {
      //_faucet = new Faucet();

      // specify an optional initial ether if you want
      _faucet = (new Faucet).value(0.5 ether)();
    }

    // call any API of Faucet as you want
  }
  ```

#### Addressing an existing instance

- **HOW**: Cast the address of an existing instance of the contract

  ```solidity
  import "Faucet.sol";

  contract Token is mortal {
    Faucet _faucet;

    constructor(address _f) {
      _faucet = Faucet(_f);
      _faucet.withdraw(0.1 ether)
    }
  }
  ```

- **Caveat**: Using addresses passed as input and casting them into specific objects is therefore
  much more dangerous than creating the contract yourself

#### Raw `call`, `delegatecall`

- These API allow us to construct a contract-to-contract call manually
- They represent the most flexible and the most dangerous mechanisms for calling other contracts

  ```solidity
  contract Token is mortal {
    constructor(address _faucet) {
      _faucet.call("withdraw", 0.1 ether);
    }
  }
  ```

- The `call` function will return `false` if there is a problem, so you can evaluate the return
  value for error handling

  ```solidity
  contract Token is mortal {
    constructor(address _faucet) {
      if !(_faucet.call("withdraw", 0.1 ether)) {
        revert("Withdrawal from faucet failed");
      }
    }
  }
  ```

- `delegatecall` runs the code of another contract
  **inside the context of the execution of the current contract**

  - It is most often used to invoke code from a library
  - The effects of `delegatecall` to non-library contract isn't promised

- A demo goes follows
  1. Deploy the dependent contract and library by 
    [deploy_called_contract_and_library.go](examples/call-delegatecall/deploy_called_contract_and_library.go)

      ```bash
      cd examples/call-delegatecall

      go run deploy_called_contract_and_library.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 10

      # Output
      deploying called contract...
      gasPrice = 20000000000
       account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
        txHash = 0x3dcfcef5f6d48635e61b79b1f895a3b8a6f336f3eaa3453c5a3eef3fa1eb5586
      done deploying called contract
      deploying called library...
      gasPrice = 20000000000
       account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
        txHash = 0xdddb2cb9e08e5429e322313577c58369192b5b3a0cf451e1ad02892b1057b991
      done deploying called library
      ```
  2. Deploy the caller contract
      ```bash
      cd examples/call-delegatecall

      # -tx: specify tx hash deploying the calledLibrary so as to query the calledLibrary's address
      go run deploy_caller.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --tx 0xdddb2cb9e08e5429e322313577c58369192b5b3a0cf451e1ad02892b1057b991 --nonce 12

      # Output
      gasPrice = 20000000000
       account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
        txHash = 0x8cdf761039c7951adcd18fc03c563aad72fc041e1a868422f0f2523edc41d135
      ```
  3. Find out the address (let's say it's `Lib`) of the deployed `calledLibrary` above
  4. Link the deployed library to the `caller` contract
     ```bash
     ./solc.sh --libraries calledLibrary:<Lib> --bin --optimize CallExamples.sol
     ```
     Replace `<Lib>` with your actual address of `calledLibrary`
     (TODO: more funny details later)
     > The `__$xxxxxx$__` in the `caller` part should disappear now
  5. Wait until all 3 contracts have been deployed on-chain successfully
  6. Call `makeCalls` by an tx with [make_calls.go](examples/call-delegatecall/make_calls.go)
      ```bash
      cd examples/call-delegatecall

      go run make_calls.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 13 --callee-tx 0x3dcfcef5f6d48635e61b79b1f895a3b8a6f336f3eaa3453c5a3eef3fa1eb5586 --caller-tx 0x8cdf761039c7951adcd18fc03c563aad72fc041e1a868422f0f2523edc41d135

      # Output
            gasPrice = 20000000000
            account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
      calledContract = 0x866f540af2999FE66F60bAc5F8B592d0e581AC17
              txHash = 0xdf7493b9c2a1fedf8e29087a7ada583e274a08e854f02caf8301e22a3b35a326
      ```
     > NOTE: The address must be padded to 32 bytes as the ABI specification
  7. After the tx above settled, checking the logging as
    [logging.go](examples/call-delegatecall/logging.go), where no panicking means ok

      ```bash
      go run logging.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --callee-tx 0x3dcfcef5f6d48635e61b79b1f895a3b8a6f336f3eaa3453c5a3eef3fa1eb5586 --caller-tx 0x8cdf761039c7951adcd18fc03c563aad72fc041e1a868422f0f2523edc41d135 --make-calls-tx 0xdf7493b9c2a1fedf8e29087a7ada583e274a08e854f02caf8301e22a3b35a326

      # Output
      checking the event emitted by _callContract.calledFunction()
      checking the event emitted by calledLibrary.calledFunction()
      check the event emitted by address(_calledContract).call(methodSig)
      check the event emitted by address(_calledContract).delegatecall(methodSig)
      ```
- Library calling always takes form of `delegatecall`

## Gas Considerations

- Gas is a resource constraining the maximum amount of computation that Ethereum will allow a
  transaction to consume
- In case of exceeded gas limit
  - An "out of gas" exception is thrown
  - The state of the contract prior to execution is restored (reverted)
  - All ether used to pay for the gas is taken as a transaction fee; it is not refunded
- Users originating txs are discouraged from calling functions that have a high gas cost
- It is in the programmer's best interest to minimize the gas cost of a contract's functions
- Some practices are introduced as follow

### Avoid Dynamically Sized Arrays

- **WHY**: Loop for seaching a target risks too much gas

### Avoid Calls to Other Contracts

### Estimating Gas Cost

1. Deploy the contract by running the [deploy.go](examples/gas-estimation/deploy.go) script
    ```bash
    go run deploy.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 14

    # Output
    gasPrice = 20000000000
     account = 0xCC6fDe13F6f662a8B752AE36f967759ECaCC82f1
      txHash = 0xf83c95251e95b5ec0c65a59030eeaf67d0e0196da1ce23c1b8aff8b9ba7cb0e9
    ```
2. After the tx is settled, run the
  [estimate_withdrawal.go](examples/gas-estimation/estimate_withdrawal.go) script should produce
  us some tips about gas

    ```bash
    go run estimate_withdrawal.go -k 5dd169f61ecdb7ac3c0b6c59e17a033f6f5c747a65a1ca83c18de108b9e5ff72 --nonce 16 --tx 0xf83c95251e95b5ec0c65a59030eeaf67d0e0196da1ce23c1b8aff8b9ba7cb0e9
    Gas price is 20000000000  wei
    Gas estimation = 31188  units
    Gas cost estimation = 623760000000000  wei
    ```

- Recommendation: Evaluate the gas cost of functions as part of your development workflow, to
  avoid any surprises when deploying contracts to the mainnet

## Conclusions
