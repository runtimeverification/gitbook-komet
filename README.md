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

### [2.1. Komet Example: Testing the `adder` Contract](guides/komet-example/)

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