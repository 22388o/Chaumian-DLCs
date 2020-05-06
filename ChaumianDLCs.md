# Chaumian 2-Party DLCs

## Abstract

Discreet Log Contracts (DLCs) were [introduced by Thaddeus Dryjaas](https://adiabat.github.io/dlc.pdf) as a way to implement oracle contracts in Bitcoin with an emphasis on privacy and scalability.
A Chaumian DLCs aims to allow the coordination for constructing these contracts in a private, trust-minimized fashion.

Note: This is an adaption of the [ZeroLink Framework](https://github.com/nopara73/ZeroLink) and uses the same underlying assumptions.

## Introduction

### What is a DLC

A Discreet Log Contract (DLC), is a contract between two or more parties based on expected oracle signatures. A typical DLC consists of 2-3 transactions.

The first step is for them to construct a Funding Transaction. This is a transaction with collateral input and a simple n-of-n multisignature output (with one key from each party)

Second, they create and sign Contract Execution Transactions (CETs), one for each possible event which spend the Funding Transaction.
These CETs have one input, spending the multisignature output from the funding transaction, and one output that is locked behind a script that requires a signature from the derived from a key from the winning party combined with the oracle's signature.

The winning party uses their key own key and oracle signature to spend the output and send it to an address they control.

### What is an adaptor signature based DLC

An adaptor signature based DLC is similiar to the DLCs described in the section above that are simpler in requiring one less transaction and only a single view, however, mostly likely require an upgrade to the bitcoin protocol for the use of adaptor signatures.

For this construction we still construct the same funding transaction, however for the CETs, the construction is different.
The CET is constructed to give the funds directly to the deserving parties based off of what the contract decrees.
The difference here is when signing, each party will adapt their signature using the oracle signature point as their adaptor point.
This way the only CET that can be executed is the one corresponding to the oracle signature point.

### What is a Blinded Signature

A blinded signature is a type of signature where the message is disguised (blinded) before it is signed.
The resulting blind signature can be publicly verified against the original message in the same way as a normal signature.

### Why use Blinded Signatures

Using a blinded signature, someone can register data related to their inputs and outputs without revealing the connection between them.
They do this by blinding one of them (in this case the outputs) and having the coordinator sign the blinded data.
Later, when they register their outputs they unblind them and use the signature they received to prove to the coordinator that they were previously registered with inputs.

## Protocol Overview for 2-Party DLCs

1. Input Registration Phase

Both Alices registers her:

- confirmed utxos as the inputs of the DLC funding transaction
- proofs - these are messages signed with the corresponding private keys, along with necessary scripts
- her desired change addresses
- blinded ContractInfo, fundingKey, toLocalCETKey, and finalAddress to the Server.

Server checks if inputs have enough coins, are unspent, confirmed, were not registered twice and that the provided proofs are valid, then signs the blinded output.
Alices unblind their signed and blinded outputs.

2. Output Registration Phase

Both Bobs register their signed ContractInfo, fundingKey, toLocalCETKey, and finalAddress to the Server.

3. Signing Phase

Sever builds the unsigned funding transaction and CETs, then gives it to Alices for signing.
The Alices then verify that these are the expected funding transaction and CETs, then signs and sends all her signatures back to the Server.
When all the Alices signatures arrive, the Servers combines the signatures and propagates the funding transaction on the network. Then passes the all of CET signatures to all of the alices.

4. Contract Execution Phase

Same as a normal DLC execution.
On contract maturity, the corresponding CET is broadcast and the winner uses the oracle signature point to spend the CET thus claiming their winnings.

### Further Notes

This construct can also be used to combine many 2 party DLCs. The only change would be that resulting funding transaction would have many 2-of-2 multisig outputs that will be used for each individual 2-party DLC.

## Protocol Overview for Adaptor Signature DLCs

1. Input Registration Phase

Many Alices each registers her:

- confirmed utxos as the inputs of the DLC funding transaction
- proofs - these are messages signed with the corresponding private keys, along with necessary scripts
- her desired change address
- blinded ContractInfo, fundingKey, and finalAddress to the Server.

Server checks if inputs have enough coins, are unspent, confirmed, were not registered twice and that the provided proofs are valid, then signs the blinded output.
Alices unblind their signed and blinded outputs.

2. Output Registration Phase

Bobs register their signed ContractInfo, fundingKey, and finalAddress to the Server.

3. Signing Phase

Sever builds the unsigned funding transaction and CETs, then gives it to Alices for signing.
The Alices then verify that these are the expected funding transaction and CETs, then signs.
Alice then adapts all of her CET signatures using the corresponding oracle signature point. She then sends all her signatures back to the Server.
When all the Alices signatures arrive, the Servers combines the signatures and propagates the funding transaction on the network. Then passes all of the CET signatures to all of the alices.

4. Contract Execution Phase

Same as a normal DLC execution.
On contract maturity, one of the particpants (or the coordinator) applies the oracle signature point to the particpants' signatures, then broadcasts.
All of the winners will have received their funds.

## Tradeoffs and Attack analysis

Same as zero link + DLC tradeoffs

ToDo: Expand further
