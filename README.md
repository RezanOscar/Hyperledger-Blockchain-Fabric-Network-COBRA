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

# Step 4: Joining the Network

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










