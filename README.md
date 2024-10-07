# Hyperledger Blockchain Fabric Network COBRA Raft Consensus Tutorial

## Overview
This repository provides a step-by-step guide to deploying a Hyperledger Fabric blockchain network using Docker. The network is designed to be a permissioned blockchain with five organizations (referred to as providers) and their respective peers, all running in a Docker-based environment.

Key Components
- 5 Providers (Organizations): Each provider has its own peer node and a Certificate Authority (CA) to manage identities.
- 1 Orderer Node: Responsible for ordering transactions and ensuring the consistency of the blockchain.
- Single Channel: All providers communicate through a single shared channel.
- LevelDB: Used as the state database for storing ledger data.
This setup aims to secure interactions among entities with a common goal while maintaining a level of trust using blockchain technology.
