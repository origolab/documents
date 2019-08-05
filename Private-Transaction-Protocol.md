# Origo Private Transaction Protocol(WIP)

## Introduction

Origo's private transaction 

### Overview

Origo's 

## Private Transactions Structure

Note: The field or structure name in this chapter may be subject to change, and may not match the name in our codebase.

Normal transactions in the Origo network, including contract call and contract creation, are in the same format as Ethereum's transaction. Private transactions extend the normal transactions by adding additional fields `private_tx` in the Transaction structure. This field is only populated for Private Transaction.

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

### Spend Description

`spends` represents all the notes being spent in the transaction. `spends` is similar to the inputs in Bitcoin transactions, but it's in an encrypted format that only the sender and receiver know the details. The following fields are included in the `SpendDescription`:

```rust
struct SpendDescription {
 cv: edwards::Point<Bls12, Unknown>,
 anchor: Fr,
 nullifier: [u8; 32],
 rk: PublicKey<Bls12>,
 zkproof: [u8; GROTH_PROOF_SIZE],
 spend_auth_sig: Option<Signature>,
}
```

`SpendDescription` includes the required information to spend a private note.

### Output Description

`outputs` represents all the new notes being created in the transaction. `outputs` is similar to the outputs in Bitcoin transactions, but also in an encrypted format. The following fields are included in the `OutputDescription`:

```rust
struct OutputDescription {
 cv: edwards::Point<Bls12, Unknown>,
 cmu: Fr,
 ephemeral_key: edwards::Point<Bls12, Unknown>,
 enc_ciphertext: [u8; 580],
 out_ciphertext: [u8; 80],
 zkproof: [u8; GROTH_PROOF_SIZE],
}
```

`OutputDescription` includes the required information to create a new note.

### Balancing Value

`balancing_value` indicates how values are transferred between public balance and private notes.

### Binding Signature

`binding_sig` is a binding signature that is used to ensure the value transfer in the whole transaction is balanced.

## Protocol

### Address Types

There're two types of address in the Origo Network: the transparent address and the private address.

The transparent address is in the same format as the Ethereum address, which is the last 160 bits of the Keccak-256 hash of the public key. It represents an account in the network.

Private addresses

### Transaction Types

Depending on the types of address used in the transaction, there are four types of Transactions.

#### Public Transaction

Both the sender and receiver use the public address. Public transactions work in the same way as Etheruem's normal transactions.

#### Shielded Transaction

A Shielded transaction transfers value from one public account to one or more private addresses.

##### Specification

The field **nonce**, **gasPrice**, **gasLimit**, **value**, **v**, **r** and **s** work in the same way as normal transactions.

* **data** should be empty since this type of transaction should not involve with contract creation or contract call.

* **to** should be empty since the value is supposed to transfer to a private address. No value will be transferred to public address.

* **balancing_value** in `PrivateTransaction` should equal to the negative of **value**. Since the value deducted from public balance will be fully transferred to private balance.

* **spends** in `PrivateTransaction` should be empty. Since we don't allow public input and private inputs being present in the same transaction.

The transaction fee will be deducted from the sender's public account balance in addition to the value. It's the same Ethereum's transaction.

#### Deshielded Transaction

A Deshielded Transaction transfers value from private addresses to one public account.

##### Specification

* The field **nonce**, **r** and **s** should be empty. Since no public input is used in this type of transaction.

#### Pure Private Transaction

A pure private transaction transfers value from private adresses to private adresses, no public addresses appear in this type of transaction.

##### Specification

* The field **nonce**, **to**, **value**, **data**, **r** and **s** should be empty. Since no public input or output should be involved. Contract creation or contract calls will not be involved as well.

* The filed **balancing_value** in `PrivateTransaction` should equal to the transaction fee for this transaction.

* Both **spends** and **outputs** in `PrivateTransaction` should not be empty.


### Transaction fees

#### Transaction Cost for Private Transaction

Private transactions invloves additional fields and requires additional computation.
Its cost should be different from normal transactions. 

#### Deducting Transaction fee

For shielded transactions, the transaction fee will be deducted from the sender's public balance.

For deshielded transactions or pure private transactions, the cost will be deducted from private inputs. 

### Signature
