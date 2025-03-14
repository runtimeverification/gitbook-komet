# Komet Example: Testing the `adder` Contract

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

## The `adder` Contract

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

### Writing The Test Contract

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

We are using the `create_contract` function from `komet.rs` for deploying a contract with a specified address and a Wasm hash. The hash represents the Wasm code of the target contract (in this case, the `adder` contract).  

The contracts passed to `init` are specified in the `kasmer.json` file, where we provide the relative path to each contract. In this example, there is only one contract—the `adder` contract—but more complex tests may require multiple contracts. Komet locates and compiles each contract, registers the Wasm module, and passes its hash to the `init` function.  

```json
{
 "contracts": [
   "../adder"
 ]
}
```

Komet will locate the `adder` contract, compile it to Wasm, and register the resulting Wasm module automatically.

If the contract requires a custom build process or is already precompiled, you can provide the path to the compiled Wasm file instead:

```json
{
  "contracts": [
    "../../target/wasm32-unknown-unknown/release/adder.wasm"
  ]
}
```


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

## Running Tests

Once the test contract is written, the next step is to compile and run it. Here's how you can execute the tests using Komet.

1. Navigate to the Test Contract Directory

After compiling the project, change directories into the `test_adder` contract folder:

```bash
cd contracts/test_adder/
```

2. Running Tests with Fuzzing

To run the tests using fuzzing (which generates random inputs for testing), use the following command:

```bash
komet test
```

After some compilation logs, you should see a progress bar:

<figure><img src=".gitbook/assets/komet-test-demo.gif" alt=""><figcaption></figcaption></figure>

This indicates that Komet discovered the `test_add` function and successfully executed the test using randomized inputs. By default, Komet runs each test 100 times. You can specify a different number of iterations using the `--max-examples` argument:

```bash
komet test --max-examples 500
```

This runs the test 500 times, allowing for more thorough fuzzing when needed.

3. Running Tests with Symbolic Execution (Proving)

To run tests with symbolic execution, which verifies the contract's behavior for all possible inputs, use the following command:

```bash
komet prove run
```

This method will run the proof for all test functions in the contract. It ensures that the property being tested holds true across all input combinations, providing thorough verification of the contract's correctness.

Additionally, you can explore more proving options by using the `--help` flag to see available commands and arguments:

```bash
$ komet prove --help
usage: komet prove [-h] [--always-allocate] [--node NODE] [--proof-dir PROOF_DIR] [--bug-report BUG_REPORT] [--extra-module EXTRA_MODULE] [--id ID] [--wasm WASM] [--directory DIRECTORY] COMMAND

positional arguments:
  COMMAND               Proof command to run. One of (run, view, view-node, remove-node)

options:
  -h, --help            show this help message and exit
  --always-allocate
  --node NODE
  --proof-dir PROOF_DIR
                        Output directory for proofs
  --bug-report BUG_REPORT
                        Bug report directory for proofs
  --extra-module EXTRA_MODULE
                        Extra module with user-defined lemmas to include for verification (which must import KASMER module).Format is <file>:<module name>.
  --id ID               Name of the test function in the testing contract
  --wasm WASM           Use a specific contract wasm file instead
  --directory DIRECTORY, -C DIRECTORY
                        The working directory for the command (defaults to the current working directory).
```

After running the proof with the `--proof-dir` option, you can use the `view` command to inspect the proof tree and examine the details of symbolic execution.
