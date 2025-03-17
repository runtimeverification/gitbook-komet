Here’s a structured documentation outline for the **Komet cheat functions** along with the explanations for each host function:

---

# Komet Cheat Functions

In addition to the standard host functions provided by the Soroban host environment, Komet includes a set of custom host functions, known as Komet cheat functions.
These cheat functions provide special abilities that normal contracts do not have and are required for testing purposes.
They allow the test contract to manipulate the blockchain state to create controlled scenarios that would be otherwise difficult or impossible to test—for example, changing the ledger timestamp to simulate time-based contract behavior. 

Below are the available Komet cheat functions:

---

## `kasmer_set_ledger_sequence`

```rust
extern "C" {
    fn kasmer_set_ledger_sequence(x: u64);
}
```

**Description**:  
This function lets us manually set the ledger sequence number, enabling tests for contract behaviors that depend on it—such as evaluating TTLs for contract data

---

## `kasmer_set_ledger_timestamp`

```rust
extern "C" {
    fn kasmer_set_ledger_timestamp(x: u64);
}
```

**Description**:  
This cheat function allows us to set the ledger timestamp. The timestamp represents the current time on the blockchain.
By manipulating the timestamp, we can simulate how contracts behave after specific periods of time.

---

## `kasmer_create_contract`

```rust
extern "C" {
    fn kasmer_create_contract(addr_val: u64, hash_val: u64) -> u64;
}

fn create_contract(env: &Env, addr: &Bytes, hash: &Bytes) -> Address {
    unsafe {
        let res = kasmer_create_contract(addr.as_val().get_payload(), hash.as_val().get_payload());
        Address::from_val(env, &Val::from_payload(res))
    }
}
```

**Description**:  
This cheat function allows us to create a contract with a specified address and Wasm hash.

---

## `kasmer_address_from_bytes`

```rust
extern "C" {
    fn kasmer_address_from_bytes(addr_val: u64, is_contract: u64) -> u64;
}

pub fn address_from_bytes<T>(env: &Env, bs: &T, is_contract: bool) -> Address
    where Bytes: TryFromVal<Env, T>
{
    use soroban_sdk::{FromVal, Val};

    let bs: Bytes = Bytes::try_from_val(env, bs).unwrap();

    unsafe {
        let res = kasmer_address_from_bytes(
            Val::from_val(env, &bs).get_payload(),
            Val::from_val(env, &is_contract).get_payload()
        );
        Address::from_val(env, &Val::from_payload(res))
    }
}
```

**Description**:  
This cheat function converts a byte array into a contract address.

---

### Conclusion

These Komet cheat functions provide the ability to perform actions that are critical for testing smart contracts in Komet. By manipulating key aspects of the blockchain environment, such as the ledger sequence, timestamp, and contract creation, we can more easily simulate and test a wide range of scenarios that would be hard to reproduce in a standard contract execution environment.
