# Chaumian DLCs using Adaptor sigs

## Introduction

TODO

- What is a DLC
- What is a Chaumian blinded signature
- Why use Chaumian sigs
- What is an adaptor sig

## Protocol Overview

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

1. Contract Execution Phase

Same as a normal DLC execution.
On contract maturity, one of the winners applies the oracle signature point to the other particpants' signatures, then broadcasts. All of the winners will have received their funds.

## Tradeoffs and Attack analysis

Same as zero link + DLC tradeoffs
