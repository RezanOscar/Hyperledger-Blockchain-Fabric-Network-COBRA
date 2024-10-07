# Hyperledger Blockchain Fabric Network COBRA Raft Consensus Tutorial

# Overview
This repository provides a step-by-step guide to deploying a Hyperledger Fabric blockchain network using Docker. The network is designed to be a permissioned blockchain with five organizations (referred to as providers) and their respective peers, all running in a Docker-based environment.

Key Components
- 5 Providers (Organizations): Each provider has its own peer node and a Certificate Authority (CA) to manage identities.
- 1 Orderer Node: Responsible for ordering transactions and ensuring the consistency of the blockchain.
- Single Channel: All providers communicate through a single shared channel.
- LevelDB: Used as the state database for storing ledger data.

This setup aims to secure interactions among entities with a common goal while maintaining a level of trust using blockchain technology.

***Please note that the blockchain cannot be used for production. For this, a complete architecture will have to be created. You can find all the necessary information on the Hyperledger. This blockchain can be used for experimentation, training, research or to do proof of work.***

# Prerequisites
Before you begin, make sure you have the following installed on your machine, to install rapidly the blockchain we will use the fabric sample binaries but with our modification to have a custom blokchain:

- Docker: Install Docker
- Docker Compose: Install Docker Compose
- Hyperledger Fabric Binaries: Download the binaries and Docker images for Fabric.
```
  - curl -sSLO https://github.com/hyperledger/fabric-ca/releases/download/v1.5.12/hyperledger-fabric-ca-linux-amd64-1.5.12.tar.gz
  - tar -xzvf hyperledger-fabric-ca-linux-amd64-1.5.12.tar.gz
```
