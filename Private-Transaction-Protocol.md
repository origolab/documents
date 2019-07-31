# Origo Private Transaction Protocol(WIP)

## Introduction
Origo's private transaction  

### Overview
Origo's 

## Private Transaction Structure
Note: The field or structure name in this chapter may be subject to change, and may not match the name in our codebase.

Normal transactions in the Origo network, including contract call and contract creation, are in the same format as Ethereum's transaction. Private transaction extends the normal transaction by adding additional fields `private_tx` in the Transaction structure. This field is only populated for Private Transaction.

```rust
struct Transaction {
    nonce: U256,
    gas_price: U256,
    gas: U256,
    action: Action,
    value: U256,
    data: Bytes,
    ...

    /// Additional field for private transaction.
    private_tx: Option<PrivateTransaction>,
}
```

The ``PrivateTransaction`` contains the following fields:

```rust
struct ConfidentialTransaction {
    spends: Vec<SpendDescription>,
    outputs: Vec<OutputDescription>,
    balancing_value: i64,
    binding_sig: [u8; 64],
}
```

`spends` represents all the notes being spent in the transaction. `outputs` represents all the new notes being created in the transaction. `spends` is similar to the inputs in Bitcoin transactions, but it's in an encrypted format that only the sender and receiver know the details. And `outputs` similar to the outputs in Bitcoin transactions, but also in an encrypted format.

`balancing_value` indicates how values are transferred between public balance and private notes. `binding_sig` is a binding signature that is used to ensure the value transfer in the whole transaction is balanced.

### Spend Description

## Protocol

### Address Types
There're two types of address in Origo Network, the transparent address and the private address.

The transparent address is in the same format as Ethereum address, which is the last 160 bits of the Keccak-256 hash of the public key. It represents an account in the network.

Private addresses 

### Transaction Types
Depending on the types of address used in the transaction, there are four types of Transactions.

#### Public Transaction
Both the sender and receiver use the public address. Public transactions work in the same way as Etheruem's normal transactions.

#### Shielded Transaction
A Shielded transaction transfers value from one public account to one or more private addresses.

##### Specification

The field ***nonce*, **gasPrice**, **gasLimit**
* **nonce:** Works the same as normal public transactions.
It should equal to the number of transactions that have been sent from this account.

* **gasPrice:**


#### Deshielded Transaction
A Deshielded Transaction transfers value from private addresses to one public account. 


#### Pure Private Transaction
A pure private transaction transfers value from private adresses to private adresses, no public addresses appear in this type of transaction.


### Transaction fees


### Signature
