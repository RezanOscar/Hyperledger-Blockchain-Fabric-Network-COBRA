# Hyperledger Blockchain Fabric Network COBRA Raft Consensus Tutorial

# Overview
This repository provides a step-by-step guide to deploying a Hyperledger Fabric blockchain network using Docker. The network is designed to be a permissioned blockchain with five organizations (referred to as providers) and their respective peers, all running in a Docker-based environment.

Key Components
- 5 Providers (Organizations): Each provider has its own peer node and a Certificate Authority (CA) to manage identities.
- 1 Orderer Node: Responsible for ordering transactions and ensuring the consistency of the blockchain.
- Single Channel: All providers communicate through a single shared channel.
- LevelDB: Used as the state database for storing ledger data.

This setup aims to secure interactions among entities with a common goal while maintaining a level of trust using blockchain technology.

> [!NOTE]
> ***Please note that the blockchain cannot be used for production. For this, a complete architecture will have to be created. You can find all the necessary information on the Hyperledger. This blockchain can be used for experimentation, training, research or to do proof of work.***


# Prerequisites
Before you begin, make sure you have the following installed on your machine, to install rapidly the blockchain we will use the fabric sample binaries but with our modification to have a custom blokchain:

- Docker: Install Docker
- Docker Compose: Install Docker Compose
- Hyperledger Fabric Binaries: Download the binaries and Docker images for Fabric.
```
      curl -sSLO https://github.com/hyperledger/fabric-ca/releases/download/v1.5.12/hyperledger-fabric-ca-linux-amd64-1.5.12.tar.gz
      tar -xzvf hyperledger-fabric-ca-linux-amd64-1.5.12.tar.gz
```
# Step 1: Architecture decision for the blockchain :

Before proceeding to the following, we must decide in advance on the architecture of our blockchain, how many organizations and peers, combination of channels and orders, what type of consensus CTO (RAFT) or Byzantine-Fault (BFT).

These elements are important in our case, our blockchain has 5 Organizations with 1 peer per Organization managed by 1 orderer and using Raft as a consensus protocol, if you wish to have a larger architecture, it will be explained at each step how to increase it.

In this tutorial, all the Organisation are named Provider

# Step 2: Create the cryptographic material for our network entities  :
After extracting the archive you will have a folder named "fabric-samples" we will create a folder in this folder which will contain all the configuration and deployment elements of our network in my case it will be called "research-network".
```
      mkdir fabric-samples/research-network
```
To generate cryptographic material for all network entities we using the cryptogen tool.

- Create the Crypto Config File: Create a config-crypto.yaml file that defines the network topology and cryptographic setup for the providers and orderer.
```
      sudo vim config-crypto.yaml
      ../bin/cryptogen generate --config config-crypto.yaml --output=config-crypto
```
This command generates all the certificates and keys for the organizations, their peers, and the orderer node.

Le fichier de config est present dans le dans le dossier research network
## Explanation of the Configuration
- OrdererOrgs: This section defines the ordering service for the blockchain network. Currently, it has one orderer node specified with the hostname orderer.
 PeerOrgs: This section lists all the providers (peers) in the network. Each provider (e.g., Provider1, Provider2) represents a different organization that will participate in the blockchain.
### Key Parameters:
- Domain: Specifies the domain for each organization.
- EnableNodeOUs: Enables the Organizational Units (OUs) that help manage certificates.
- Template Count: Defines the number of peers for each organization. By default, each provider has one peer.
- Users Count: Specifies the number of admin users to generate for each provider.

> [!TIP]
> To increase the number of providers in your network, you simply need to add more entries under the PeerOrgs section of the crypto-config.yaml file. If you want to add more peers to an existing provider, modify the Count value in the Template section for that provider. After making changes, regenerate the cryptographic material using the same cryptogen command to reflect the updated network configuration.


# Step 3: Create the Genesis Block and Channel Configuration
- Create the Configuration File: Create a configtx.yaml file to define the network's orderer and channel details.
- Generate the Genesis Block for the Orderer:
```
      ../bin/configtxgen -profile OrdererGenesis -outputBlock ./channel-artifacts/genesis.block -channelID channelorderergenesis
```
- Generate Channel Configuration:
```
      ../bin/configtxgen -profile ChannelCoop -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID channelcoop
```
- Generate Channel Configuration:
```
      ../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider1Anchor.tx -channelID channelcoop -asOrg Provider1MSP
      ../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider2Anchor.tx -channelID channelcoop -asOrg Provider2MSP
      ../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider3Anchor.tx -channelID channelcoop -asOrg Provider3MSP
      ../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider4Anchor.tx -channelID channelcoop -asOrg Provider4MSP
      ../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider5Anchor.tx -channelID channelcoop -asOrg Provider5MSP
```

# Step 4: Docker Configuration
- Create Docker Configuration Files: Create the following Docker configuration files in the research-network/docker directory:
  - docker-compose.yaml: Defines the services for peers, orderer, and other network components.
  - peer.yaml: Configuration specific to peer nodes.
- Set the Docker Environment:

```
      echo COMPOSE_PROJECT_NAME=net > .env
```
- Start the Docker Containers:

```
      sudo docker-compose -f docker-compose-cli.yaml up -d
```



> [!IMPORTANT]
> Restart Docker Containers if Necessary: If there is an error, stop all containers and remove them:
>```
>      docker stop $(docker ps -aq)
>      docker rm $(docker ps -aq)
>      docker volume prune
>      docker network prune
>      docker images
>```

