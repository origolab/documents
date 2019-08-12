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
struct PrivateTransaction {
 version: u32,
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

`balancing_value` indicates how values are transferred between public balance and private notes. A positive `balancing_value` transfers value from private input to public balance; a negative `balancing_value` transfers value from public account to private output.

### Binding Signature

`binding_sig` is a binding signature that is used to ensure the value transfer in the whole transaction is balanced.

## Protocol

### Address Types

There're two types of address in the Origo Network: the transparent address and the private address.

The transparent address is in the same format as the Ethereum address, which is the last 160 bits of the Keccak-256 hash of the public key. It represents an account in the network.

Private addresses

### Transaction Types

Depending on the types of address used in the transaction, there are four types of Transactions, and a special private contract call or creation.

#### Public Transaction

Both the sender and receiver use the public address. Public transactions work in the same way as Etheruem's normal transactions.

#### Shielded Transaction

A Shielded transaction transfers value from one public account to one or more private addresses.

##### Specification

* **nonce**, **gasPrice**, **gasLimit**, **value**, **v**, **r** and **s** work in the same way as normal transactions.

* **to** should be empty since the value is supposed to transfer to a private address. No value will be transferred to the public address.

* **data** should be empty since this type of transaction should not involve with contract creation or contract call.

* **spends** in `PrivateTransaction` should be empty. Since we don't allow public input and private inputs being present in the same transaction.

* **outputs** in `PrivateTransaction` should not be empty.

* **balancing_value** in `PrivateTransaction` should equal to the negative of **value**. Since the value deducted from public balance will be fully transferred to private balance.
The transaction fee will be deducted from the sender's public account balance in addition to the value. It's the same Ethereum's transaction.

#### Pure Private Transaction

A pure private transaction transfers value from private adresses to private adresses, no public addresses appear in this type of transaction.

##### Specification

* **nonce**, **to**, **value**, **data**, **r** and **s** should be empty. Since no public input or output should be involved. Contract creation or contract calls will not be involved as well.

* **gasPrice** works the same, it represents the number of Wei paid per gas unit.

* **gasLimit** has no refund. Refer the [Transaction fee](#Transaction-Fees) for more details.

* **balancing_value** equals to gasPrice * gasLimit.

* Both **spends** and **outputs** in `PrivateTransaction` should not be empty.

#### Deshielded Transaction

A Deshielded Transaction transfers value from private addresses to one public account.

##### Specification

* **nonce**, **r** and **s** should be empty. Since no public input is involved.

* **gasLimit** has no refund.

* **to**, **value**, **data**, **gasPrice** works the same as normal transaction.

* **balancing_value** equals to value + gasPrice * gasLimit.

* **spends** in `PrivateTransaction` should not be empty. **outputs** is used to send change.

#### Private Contract Call/Creation

A private contract creation or contract call allows users to create a smart contract, or call a smart contract anonymously, without revealing its public account. The public sender will not be represented. Instead, the gas and value will be paid from private inputs.

This type of transaction will not be supported in the first version of the Origo network.
We preserved its encoding for future extension.

##### Specification

* **gasPrice**, **to**, **value**, **data** work the same as Ethereum's contract call or creation.

* **nonce**, **r** and **s** should be empty. Since no public account is involved.

* **gasLimit** has no refund.

* **balancing_value** equals to value + gasPrice * gasLimit.

* **spends** in `PrivateTransaction` should not be empty. **outputs** is used to send change.

#### Summary

Here's a summary of the specification for each field for different transaction types.

||Public to Public|Public to contract/creation|Shielded|Pure Private|Deshield|Private Contract Call/Creation|
|---|---|---|---|---|---|---|
|Nonce|sender nonce|sender nonce|sender nonce|empty|empty|empty|
|gasPrice|wei/gas|wei/gas|wei/gas|wei/gas|wei/gas|wei/gas|
|gasLimit|max gas|max gas|max gas|gas cost|gas cost|gas cost|
|to|receiver address|contract address, empty for contract creation|empty|empty|receiver address|contract address, |empty for contract creation(data shouldn't be empty)|
|value|value if any|value if any|value|0|value|value if any|
|data|empty?|data if any|empty|empty|empty|data if any|
|v|id|id|id|id|id|id|
|r, s|signature|signature|signature|empty|empty|empty|
|spends|NA|NA|empty|private input|private input|private input|
|outputs|NA|NA|private ouput|private ouput|change or empty|change or empty|
|balancing value|NA|NA|-value|gasLimit*price|value + gasLimit*price|value + gasLimit*price|

### Transaction Fees

#### Gas Cost for Private Transaction

Private transactions invloves additional fields and requires additional computation.
Its cost should be different from normal transactions. The gas cost should equals to:

g<sub>private_tx</sub> = g<sub>base</sub> + g<sub>spend</sub>*num_of_spends + g<sub>output</sub>*num_of_outputs.

The exact value of g<sub>base</sub>, g<sub>spend</sub> and g<sub>output</sub> will be specified later.

#### Deduct Transaction Fee

For shielded transactions, the transaction fee will be deducted from the sender's public balance. **gasLimit** is used to specify the maximum amount of gas that should be used in executing this transaction.
It's paid upfront and the unused gas will be refunded to the sender's account.

For deshielded transactions or pure private transactions, the cost will be deducted from private inputs. One major difference is that it's impossible to refund the unused gas back to private inputs. So **gasLimit** specifies the amount of gas paid for executing the transaction. If it exceeds the required gas amount, no refund will be issued.

### Signature

Origo's private transactions utilize three signature schemas:

* For transactions that involve a public sender account, ECDSA signature based on the SECP-256k1 curve is used to sign the transaction.

* Spend Authorization Signature is used to sign authorizations of spending private notes.

* Binding Signature is used to enforce the balance of Spend transfers and Output transfers, and to prevent their replay across transactions.

#### Transaction Hash for Signature(SigHash)

Signatures and/or non-interactive proofs associated with transaction inputs are used to authorize spending. To prevent these signatures or proofs being replayed in a different transaction, they should be bind to the transaction for which they are intended. This is done by hashing the transaction and use the transaction hash for the signature schemas mentioned above.

As an example, Ethereum's transaction signature is signed over the hash of the rlp encoding of (nonce, gasPrice, gasLimit, to, value, data, v). Such that those fields can not be modified to create another transaction with the same valid signature. For normal transactions that don't involve the private transaction field, the hash for signature is computed in the same way as Ethereum.

For transactions that involve private inputs or outputs, the fields in `PrivateTransaction`(except for `spend_auth_sig` and `binding_sig`) will be included in the transaction hash for Signature. So the `sigHash` will be computed as:

```rust
fn hash(tx: &Transaction, chain_id: Option<u64>) -> H256 {
 let mut s = RlpStream::new();

 let offset = if tx.is_private() { 1 } else { 0 };
 s.begin_list(if chain_id.is_none() { 6 } else { 9 } + offset);
 s.append(&tx.nonce);
 s.append(&tx.gas_price);
 s.append(&tx.gas);
 s.append(&tx.action);
 s.append(&tx.value);
 s.append(&tx.data);
 if tx.is_private() {
 // spend_auth_sig and binding_sig is appended as "".
 s.append(&tx.private_tx);
 }
 if let Some(n) = chain_id {
 s.append(&n);
 s.append(&0u8);
 s.append(&0u8);
 }
 keccak(s.as_raw())
}
```

**Note:** The sigHash computation is different from the [method](https://github.com/zcash/zips/blob/master/zip-0243.rst) used by Zcash. In their approach, each field is hashed by BLAKE2b with personalization, and then all the hashed values are then combined and hashed again. We should examine if there's any security flaw.
