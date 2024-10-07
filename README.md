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

> [!CAUTION]
> All the configuration files that will be described later are in the researhc folder. It is important to respect the architecture of the folder so that everything works.


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
This step is crucial in setting up the Hyperledger Fabric network. The genesis block and channel configuration define the structure of your blockchain network and how different nodes (Orderer and Peers) interact with each other.

**Genesis Block:** This is the first block of the blockchain that initializes the Orderer service. It contains the configuration of the Orderer and the setup for the initial network.
**Channel Configuration:** Channels are used to facilitate private and secure communication between a subset of network members. Each channel has its own ledger and smart contract instances.

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

This file defines the structure of the blockchain network, specifying the organizations, policies, and capabilities. It is used by the configtxgen tool to generate the genesis block and other artifacts for channel creation.

Key Sections in configtx.yaml:
- Organizations: Defines the details for each organization (Orderer and Providers) in the network, including their identities, policies, and anchor peers.
- Capabilities: Specifies the feature set available for the channel and application.
- Application: Defines the settings for the chaincode lifecycle and policies for reading, writing, and validating transactions.
- Orderer (Raft Configuration): Defines the settings for the ordering service, which ensures the order of transactions in the blockchain.
- Channel: Configures the policies for creating and managing channels in the network.
- Profiles: Contains different configuration profiles that can be used to generate the genesis block and channel artifacts.

> [!TIP]
> To increase the number of providers in your network, to increase the number of providers in your network, you need to make changes in the configtx.yaml file as well as the crypto-config.yaml file to ensure that the new providers are properly integrated.
> Add a New Provider in the crypto-config.yaml. Follow the same format used for the existing providers to add a new provider, say Provider7.
> Modify the configtx.yaml File: Under the Organizations section, add a new entry for the new provider and Update the Profiles Section, add the new provider to both the Consortium and Channel profiles to ensure it is included in the network
> Finally to scale up the architecture:
> - Increase the Orderer Nodes: If you need to scale the Orderer service, you can add additional orderer nodes under the OrdererOrgs section by specifying more hostnames.
> - Add More Channels: To create a new channel, add its definition in the Profiles section and use configtxgen to generate its transaction file.
> - Adjust Batch Size for Higher Throughput: Increase the MaxMessageCount and PreferredMaxBytes in the Orderer configuration to handle larger volumes of transactions.

# Step 4: Docker Configuration
In this step, we will configure Docker to set up the Hyperledger Fabric network environment. Docker Compose is used to define and manage multiple containers, including the Orderer and Peer nodes, which form the backbone of our blockchain network. The Docker configuration also includes settings for the CLI container that will be used to interact with the network.

Docker Compose files provide the environment setup for each component in the Hyperledger Fabric network. They specify the services, environment variables, and ports for each node. This setup allows you to deploy a distributed blockchain network across multiple nodes in a consistent and replicable manner.

## Key Components of the Docker Configuration
### Docker Compose Configuration for Peer Nodes
- Objective: Defines the environment and settings for each peer node in the network.
Environment Variables:
- Specifies important parameters like logging level, TLS certificates, and node identities.
- Ensures the peer nodes use secure communication by setting up TLS (Transport Layer Security).
- Volume Mapping: Maps the local directories to the Docker container to store the cryptographic material, configuration files, and ledger data.

### Docker Compose Configuration for the Orderer Node
- Objective: Sets up the Orderer node, which is responsible for maintaining the order of transactions in the blockchain.
Orderer Environment Settings:
- Defines the location of the genesis block and TLS settings to ensure secure communication.
- Volume Mapping: Stores the Orderer’s configuration and data files securely in the Docker environment.

### CLI Container Configuration
- Purpose: The CLI (Command Line Interface) container allows you to interact with the blockchain network for operations like chaincode installation, querying, and invoking transactions.
Environment Variables:
- Includes settings related to the peer address, MSP (Membership Service Provider), and the file paths for cryptographic keys.
- Volume Mapping: Maps the local directories containing chaincode and channel artifacts to the CLI container, enabling easy access to required files.

- Create Docker Configuration Files: Create the following Docker configuration files in the research-network/docker directory:
  - docker-compose.yaml: Defines the services for peers, orderer, and other network components.
  - peer.yaml: Configuration specific to peer nodes.
  - docker-compose-cli.yaml: To setup the docker, create in the Racine
```
	vim docker-compose-cli.yaml
	mkdir research-network/docker
	vim docker-compose.yaml
	vim peer-docker.yaml
```
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


> [!TIP]
> Scaling the Network: Adding More Providers To add more providers or peer nodes to your existing setup, follow these steps:
> Update the peer-docker.yaml File
> - Add a new service definition for the additional peer node, making sure to update the environment variables with the new peer’s details.
> - Modify the docker-compose.yaml File
> Include the new peer service in the overall Docker Compose setup.
> - Ensure that the network settings and ports are correctly mapped to avoid conflicts with existing services.
> - Adjust Volumes and Networks
> Create new volumes for the additional peers to store their data.
> - Connect the new peer nodes to the existing Docker network for seamless communication.


# Step 5: Joining the Network
This step involves connecting each peer to the blockchain network, ensuring that they can interact with each other and the Orderer node to form a secure, decentralized network. You will use Docker commands to link the peers to the network and update their configurations to facilitate communication within the channel.
- For each peer in your network, you need to run specific Docker commands to set up the environment variables and connect to the blockchain network.
- Open a separate terminal window for each peer and execute the commands.
- The commands will set up the CORE_PEER_LOCALMSPID, the TLS root certificate, and other necessary configuration details for each peer node.

## Command:
- Execute Docker Commands for Each Peer: Run the following commands for each peer to join them to the network. Open separate terminal windows for each peer.
```
      docker exec -e "CORE_PEER_LOCALMSPID=Provider1MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro1.research-network.com/peers/peer0.pro1.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro1.research-network.com/users/Admin@pro1.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro1.research-network.com:7051" -it cli bash
      docker exec -e "CORE_PEER_LOCALMSPID=Provider2MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro2.research-network.com/peers/peer0.pro2.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro2.research-network.com/users/Admin@pro2.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro2.research-network.com:7051" -it cli bash
      docker exec -e "CORE_PEER_LOCALMSPID=Provider3MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro3.research-network.com/peers/peer0.pro3.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro3.research-network.com/users/Admin@pro3.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro3.research-network.com:7051" -it cli bash
      docker exec -e "CORE_PEER_LOCALMSPID=Provider4MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro4.research-network.com/peers/peer0.pro4.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro4.research-network.com/users/Admin@pro4.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro4.research-network.com:7051" -it cli bash
      docker exec -e "CORE_PEER_LOCALMSPID=Provider5MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro5.research-network.com/peers/peer0.pro5.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro5.research-network.com/users/Admin@pro5.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro5.research-network.com:7055" -it cli bash
```
- Add environement varible in all the terminal:
```
      export ORDERER_CA=/opt/gopath/fabric-samples/research-network/crypto-config/ordererOrganizations/research-network.com/orderers/orderer.research-network.com/msp/tlscacerts/tlsca.research-network.com-cert.pem
```
- Create and Join the Channel: In one terminal:
```
      peer channel create -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/channel.tx --tls --cafile $ORDERER_CA
```
  - In each peer's terminal, join them to the channel: 
```
      peer channel join -b channelcoop.block --tls --cafile $ORDERER_CA
```
- Update Anchor Peers: Update the anchor peers for each provider:
```
      peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider1Anchor.tx --tls --cafile $ORDERER_CA
      peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider2Anchor.tx --tls --cafile $ORDERER_CA
      peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider3Anchor.tx --tls --cafile $ORDERER_CA
      peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider4Anchor.tx --tls --cafile $ORDERER_CA
      peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider5Anchor.tx --tls --cafile $ORDERER_CA
```

> [!IMPORTANT]
> Terminal Management: It's important to open a separate terminal for each peer to manage them independently. This practice helps in monitoring each node's behavior and catching any issues that may arise during the setup.
> Troubleshooting: If any peer fails to join the channel, recheck the environment variables and the certificate paths. Restarting the Docker containers may also help resolve connectivity issues.
> Scalability: To add more peers to the network, replicate the steps for each new peer node, making sure to adjust the environment variables and port mappings accordingly.

# Step 6: Verifying the Network Setup
- Check Docker Logs: Use the following command to check the logs of any container:
```
      docker compose logs <container-id>
```
- Verify the Network Setup: Ensure that all peers and the orderer are up and running without issues.

> [!WARNING]
> If you encounter issues with Docker containers not starting or joining the network, use the following commands to clean up:
>```
>      docker stop $(docker ps -aq)
>      docker rm $(docker ps -aq)
>      docker volume prune
>      docker network prune
>```


# Troubleshooting

- 2024-07-09 14:16:47.340 UTC 0003 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
Error: got unexpected status: BAD_REQUEST -- error validating channel creation transaction for new channel 'channelcoop', could not successfully apply update to template configuration: error authorizing update: error validating DeltaSet: policy for [Group]  /Channel/Application not satisfied: implicit policy evaluation failed - 0 sub-policies were satisfied, but this policy requires 1 of the 'Admins' sub-policies to be satisfied

**Solution : Check if the variables put in the docker exec correspond to what is present in the configtx.yaml file**

- 2024-07-10 10:31:14.775 UTC 0005 PANI [orderer.common.server] Main -> Failed validating bootstrap block: cannot enable channel capabilities without orderer support first panic: Failed validating bootstrap block: cannot enable channel capabilities without orderer support first         [orderer.common.server] Main -> Failed validating bootstrap block: cannot enable channel capabilities without orderer support first panic: Failed validating bootstrap block: cannot enable channel capabilities without orderer support first goroutine 1 [running]: go.uber.org/zap/zapcore.CheckWriteAction.OnWrite(0x0?, 0x0?, {0x0?, 0x0?, 0xc000316940?})

**Solution: You need to add Capabilities: <<: \*OrdererCapabilities in the orderer section**

- Failed validating bootstrap block: initializing channelconfig failed: could not create channel Orderer sub-group config: Orderer Org OrderingService cannot contain endpoints value until V1_4_2+ capabilities have been enabled
panic: Failed validating bootstrap block: initializing channelconfig failed: could not create channel Orderer sub-group config: Orderer Org OrderingService cannot contain endpoints value until V1_4_2+ capabilities have been enabled
**Solution:**
```
      Capabilities:
	    Channel: &ChannelCapabilities
	        V2_0: true
	    Orderer: &OrdererCapabilities    
	        V2_0: true
	    Application: &ApplicationCapabilities
	        V2_0: true
```
	
- 0003 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
- Error: proposal failed (err: rpc error: code = Unknown desc = error validating proposal: access denied: channel [] creator org unknown, creator is malformed)**
- Error in the docker-compose.yaml or configtx file correctly linked in particular Environment Variable Error: failed to retrieve broadcast client: failed to load config for OrdererClient: unable to load orderer.tls.rootcert.file: open /etc/hyperledger/ fabric/--channelID: no such file or directory**

**Solution: export ORDERER_CA=/opt/gopath/fabric-samples/research-network/crypto-config/ordererOrganizations/research-network.com/orderers/orderer.research-network.com/msp/tlscacerts/tlsca.research-network.com-cert.pem**

- Failed to query chaincode: Failed to get endorsing peers: error getting channel response for channel [channelcoop]: Discovery status Code: (11) UNKNOWN. Description: error received from Discovery Server: failed constructing descriptor for chaincodes:<name:"test_4" >

**Solution: check if the Smart Contract name matches the committed one with this command "peer lifecycle chaincode querycommitted -C channelcoop"**

- failed: sanitizeCert failed the supplied identity is not valid: x509: certificate signed by unknown authority

**Solution : Switch to GO version lower than 1.18.5 "go version"**



# All comand use to create the network:
***Below are all the commands to use in raw form if you want to automate the process.***
```
	mkdir research-network
	vim crypto-config.yaml
	../bin/cryptogen generate --config crypto-config.yaml --output=crypto-config
	vim configtx.yaml
	../bin/configtxgen -profile OrdererRaft -outputBlock ./channel-artifacts/genesis.block -channelID channelorderer
	../bin/configtxgen -profile ChannelCoop -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID channelcoop

	../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider1Anchor.tx -channelID channelcoop -asOrg Provider1MSP
	../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider2Anchor.tx -channelID channelcoop -asOrg Provider2MSP
	../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider3Anchor.tx -channelID channelcoop -asOrg Provider3MSP
	../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider4Anchor.tx -channelID channelcoop -asOrg Provider4MSP
	../bin/configtxgen -profile ChannelCoop -outputAnchorPeersUpdate ./channel-artifacts/Provider5Anchor.tx -channelID channelcoop -asOrg Provider5MSP

	mkdir docker
	cd docker/

	vim docker-compose.yaml
	vim peer-docker.yaml
	cd ..
	vim docker-compose-cli.yaml
	echo COMPOSE_PROJECT_NAME=net > .env
	docker-compose -f docker-compose-cli.yaml up -d

**One terminal for each command :**
	docker exec -e "CORE_PEER_LOCALMSPID=Provider1MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro1.research-network.com/peers/peer0.pro1.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro1.research-network.com/users/Admin@pro1.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro1.research-network.com:7051" -it cli bash
	docker exec -e "CORE_PEER_LOCALMSPID=Provider2MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro2.research-network.com/peers/peer0.pro2.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro2.research-network.com/users/Admin@pro2.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro2.research-network.com:7051" -it cli bash
	docker exec -e "CORE_PEER_LOCALMSPID=Provider3MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro3.research-network.com/peers/peer0.pro3.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro3.research-network.com/users/Admin@pro3.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro3.research-network.com:7051" -it cli bash
	docker exec -e "CORE_PEER_LOCALMSPID=Provider4MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro4.research-network.com/peers/peer0.pro4.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro4.research-network.com/users/Admin@pro4.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro4.research-network.com:7051" -it cli bash
	docker exec -e "CORE_PEER_LOCALMSPID=Provider5MSP" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro5.research-network.com/peers/peer0.pro5.research-network.com/tls/ca.crt" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/fabric-samples/research-network/crypto-config/peerOrganizations/pro5.research-network.com/users/Admin@pro5.research-network.com/msp" -e "CORE_PEER_ADDRESS=peer0.pro5.research-network.com:7055" -it cli bash


**In all terminal :**
	export ORDERER_CA=/opt/gopath/fabric-samples/research-network/crypto-config/ordererOrganizations/research-network.com/orderers/orderer.research-network.com/msp/tlscacerts/tlsca.research-network.com-cert.pem

**One terminal :**
	peer channel create -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/channel.tx --tls --cafile $ORDERER_CA

**In all terminal :**
	peer channel join -b channelcoop.block --tls --cafile $ORDERER_CA
	peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider1Anchor.tx --tls --cafile $ORDERER_CA
	peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider2Anchor.tx --tls --cafile $ORDERER_CA
	peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider3Anchor.tx --tls --cafile $ORDERER_CA
	peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider4Anchor.tx --tls --cafile $ORDERER_CA
	peer channel update -o orderer.research-network.com:7050 -c channelcoop -f /opt/gopath/fabric-samples/research-network/channel-artifacts/Provider5Anchor.tx --tls --cafile $ORDERER_CA
```











