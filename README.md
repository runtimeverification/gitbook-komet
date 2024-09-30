# Komet Documentation

## 1. Overview

### 1.1 Komet

Komet is a formal verification and fuzzing tool specifically designed for Soroban smart contracts, built on top of Runtime Verification's K Semantics framework. It allows developers to write property tests in Rust, the same language used for Soroban contracts, and then test these properties across a wide range of inputs using fuzzing. More importantly, Komet provides formal verification by symbolically executing the contracts, ensuring their correctness across all possible inputs, providing the highest level of assurance for contract safety and functionality.

Fuzzing allows developers to automatically test smart contracts by generating large sets of randomized inputs, exploring edge cases and finding vulnerabilities in contract logic. Combined with formal verification, Komet enables developers to go beyond traditional testing by symbolically executing contracts—proving that they behave as expected in every possible scenario. This makes Komet a powerful tool for developers aiming to secure their smart contracts, particularly in the high-stakes environment of decentralized finance (DeFi), where millions of dollars can be at risk from bugs or exploits.

Built with the foundation of KWasm, a WebAssembly semantics framework, Komet seamlessly integrates with the Soroban ecosystem. By offering Rust-based property testing and verification, it helps Soroban developers maintain security and reliability within their smart contracts while leveraging the broader Rust and WebAssembly toolchain. This combination of ease of use and rigorous testing methodologies ensures that developers can write safer contracts with fewer vulnerabilities.

### 1.2 The Significance

#### Why do you need formal verification?

While fuzzing is a powerful testing technique, and a considerable step up from the plain unit testing, it still has some limitations that motivate the need for a complementary symbolic execution of the test suite. Due to pseudo-random input generation, fuzzing struggles to generate input values for complex or nested conditions. Symbolic execution, systematically explores all feasible code paths by using symbolic variables as input, tracking path conditions. This provides a more comprehensive coverage.

In addition to this, symbolic execution can automatically derive and check postconditions, providing stronger guarantees on the correctness of the program. These complementary approaches can be used to identify a wider range of bugs and vulnerabilities, leading to more reliable software.

By installing Komet, you unlock the capability to perform property verification! This is a big step up in assurance from property testing, but is more computationally expensive, and often requires manual intervention.

### 1.3 Installation

The easiest way to install Komet is via the kup package manager. Follow the steps below to install kup and then Komet:

1. Install `kup` by running the following command:

```bash
bash <(curl https://kframework.org/install)
```

2. Once `kup` is installed, you can install Komet with this command:

```bash
kup install komet
```

3. After the installation is complete, verify it by checking the help menu:

```bash
komet --help
```

## 2. Guides

### 2.1. Komet Example: Testing the `adder` Contract

(Source: https://github.com/runtimeverification/komet-demo)

To illustrate how Komet can be used to test Soroban contracts, let's look at a simple example. We'll be working with a basic contract called adder, which features a single function that adds two numbers and returns their sum. In Komet, we write our tests as contracts that interact with the contract we want to test. For this example, we will create a test_adder contract to verify the behavior of the adder contract.

The project structure for this example looks like this:

```
.
├── contracts
│   ├── adder
│   │   ├── src
│   │   │   └── lib.rs
│   │   └── Cargo.toml
│   └── test_adder
│       ├── src
│       │   ├── lib.rs
│       │   └── komet.rs
│       ├── Cargo.toml
│       └── kasmer.json
├── Cargo.toml
└── README.md
```

#### The `adder` Contract

The adder contract is a simple, stateless contract with a single endpoint, add. This function takes two numbers as input and returns their sum. The result is returned as a u64 to avoid overflows. Since the contract doesn't maintain any internal state or use storage, its behavior is straightforward and purely based on the inputs provided.

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, Env};

#[contract]
pub struct AdderContract;

#[contractimpl]
impl AdderContract {
   pub fn add(_env: Env, first: u32, second: u32) -> u64 {
      first as u64 + second as u64
   }
}
```

### 2.2 Writing The Test Contract

Test contracts typically have an init function for setting up the initial state, such as deploying contracts or preparing the blockchain environment. They also include functions with names starting with test_ to define properties and run test cases against the contract being tested. Test contracts have special abilities that normal contracts do not, provided by our framework through custom WebAssembly hooks. These hooks, declared as extern functions in komet.rs, enable advanced operations like contract creation and state manipulation.

#### Setting the Initial State: The init Function

In the context of testing the adder contract, the init function is specifically responsible for deploying the adder contract and saving its address within the test contract's storage.

```rust
#[contract]
pub struct TestAdderContract;

#[contractimpl]
impl TestAdderContract {
   pub fn init(env: Env, adder_hash: Bytes) {
       let addr_bytes = b"adder_ctr_______________________";
       let adder = komet::create_contract(&env, &Bytes::from_array(&env, addr_bytes), &adder_hash);
       env.storage().instance().set(&ADDR_KEY, &adder);
   }

   // other functions
}
```

We are using the create_contract function from komet.rs for deploying a contract with a specified address and a wasm hash. The hash represents the Wasm code of the target contract (in this case, the adder contract). The wasm hash provided to the init function is derived from the kasmer.json file, which contains a relative path to the compiled adder contract. This file enables Komet to locate the wasm file, register the Wasm module, and pass its hash to the init function.

```json
{
 "contracts": [
   "../../target/wasm32-unknown-unknown/release/adder.wasm"
 ]
}
```

Once the init function has successfully completed, all subsequent test cases are executed based on this predefined initial state, ensuring consistency and allowing tests to be performed under controlled conditions.

#### Defining Contract Properties: test endpoints

In Komet, test cases are defined as contract endpoints with names starting with the test_ prefix. These endpoints specify properties of the contract being tested and return a boolean to indicate whether the test passed or failed.

For instance, in the test_adder contract, the test_add function verifies the adder contract's behavior by using its address—set up in the init function—to invoke the add method and check whether the adder contract correctly computes the sum of two numbers.

```rust
impl TestAdderContract {
   // Initialization code...
  
   pub fn test_add(env: Env, x: u32, y: u32) -> bool {
       // Retrieve the address of the `adder` contract from storage
       let adder: Address = env.storage().instance().get(&ADDR_KEY).unwrap();
      
       // Create a client for interacting with the `adder` contract
       let adder_client = adder_contract::Client::new(&env, &adder);

       // Call the `add` endpoint of the `adder` contract with the provided numbers
       let sum = adder_client.add(&x, &y);
      
       // Check if the returned sum matches the expected result
       sum == x as u64 + y as u64
   }
}
```

### 2.3 Running Tests

Once the test contract is written, the next step is to compile and run it. Here's how you can execute the tests using Komet.

1. Compile the Project

Before running any tests, compile the project from the workspace's root directory:

```bash
soroban contract build
```

This command will build both the `adder` and `test_adder` contracts.

2. Navigate to the Test Contract Directory

After compiling the project, change directories into the `test_adder` contract folder:

```bash
cd contracts/test_adder/
```

3. Running Tests with Fuzzing

To run the tests using fuzzing (which generates random inputs for testing), use the following command:

```bash
komet test
```

After some compilation logs, you should see an output like this:

```
Processing contract: test_adder
Discovered 1 test functions:
	- test_add

  Running test_add...
	Test passed.
```

This indicates that Komet discovered the `test_add` function and successfully executed the test using randomized inputs.

4. Running Tests with Symbolic Execution (Proving)

To run tests with symbolic execution, which verifies the contract's behavior for all possible inputs, use the following command:

```bash
komet prove run
```

This method will run the proof for all test functions in the contract. It ensures that the property being tested holds true across all input combinations, providing thorough verification of the contract's correctness.

Additionally, you can explore more proving options by using the `--help` flag to see available commands and arguments:

```bash
$ komet prove --help
usage: komet prove [-h] [--wasm WASM] [--proof-dir PROOF_DIR] [--id ID] COMMAND

positional arguments:
  COMMAND           	Proof command to run. One of (run, view)

options:
  -h, --help        	show this help message and exit
  --wasm WASM       	Prove a specific contract wasm file instead
  --proof-dir PROOF_DIR
                    	Output directory for proofs
  --id ID           	Name of the test function in the testing contract
```

After running the proof with the `--proof-dir` option, you can use the `view` command to inspect the proof tree and examine the details of symbolic execution.

## 3. Real-World Demo

This section provides a practical demonstration of Komet's capabilities, showcasing how it can be used for fuzzing and formal verification of Soroban smart contracts. By following this real-world example, developers can gain an understanding of how Komet can be integrated into their workflows to secure smart contracts efficiently.

### 3.1 Demo Overview

In this demo, we will use Komet to fuzz and formally verify a function within the fxDAO smart contract. The video walkthrough covers the process of writing property-based tests in Rust and running them through Komet's symbolic execution engine.

Komet is built on K Semantics, which allows it to execute and verify WebAssembly (Wasm) code. The demo demonstrates how fuzzing is used to generate random test inputs, testing the robustness of the contract, and how formal verification ensures correctness across all possible inputs.

### 3.2 Video Walkthrough

Watch the demo video below for a step-by-step guide to fuzzing and verifying the fxDAO smart contract using Komet. This video highlights key features like:

- Writing property-based tests in Rust
- Setting up fuzzing for edge case detection
- Running formal verification to guarantee security and correctness

Demo Video: [https://www.youtube.com/watch?v=76VD0aKPXGE](https://www.youtube.com/watch?v=76VD0aKPXGE)