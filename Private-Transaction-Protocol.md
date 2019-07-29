# Origo Private Transaction Protocol(WIP)

## Introduction
Origo's private transaction  

### Overview
Origo's 

## Private Transaction Structure
Note: The field or structure name in this chapter may be subject to change, and may not match the name in our codebase.

Normal transactions in the Origo network, including contract call and contract creation, are in the same format as Ethereum's transaction. Private transaction extends the normal transaction by adding additional fields `private_tx` in the Transaction structure. This field is only populated for Private Transaction.
```
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
```
struct ConfidentialTransaction {
    spends: Vec<SpendDescription>,
    outputs: Vec<OutputDescription>,
    balancing_value: i64,
    binding_sig: [u8; 64],
}
```
`spends` represents all the notes being spent in the transaction. `outputs` represents all the new notes being created in the transaction. `spends` is similar to the inputs in Bitcoin transactions, but it's in an encrypted format that only the sender and receiver know the details. And `outputs` similar to the outputs in Bitcoin transactions, but also in an encrypted format.

`balancing_value` indicates how values are transferred between public balance and private notes.  `binding_sig` is a binding signature that is used to ensure the value transfer in the whole transaction is balanced.

### Spend Description

## Protocol

### Address Types
### Transaction Types




### Signature
