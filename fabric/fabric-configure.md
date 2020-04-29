# Install Prerequisites
## git, curl (take git as an example. for curl, just replace *git* with *curl*)

In ubuntu, install git with apt

`sudo apt install git`

Check git installation

`git --version`

## docker and docker compose

Install docker engine with the command

`sudo apt install docker.io`

docker.io is the name of deb package of docker engine for ubuntu.

Check docker version, and make sure it is greater than 17.06.2.

`docker --version`

Keep docker running with systemd.

`sudo systemctl start docker`

Automatically start docker daemon when system starts.

`sudo systemctl enable docker`

(optional) Add user to docker group to manage docker with non-root user.

```
# create a group named docker
sudo groupadd docker

# add your user into the docker group
sudo usermod -aG docker $USER

# activate the change
newgrp docker
# If this does not work, try to end current session and start a new one, or
# if work in vm, try to restart the machine to make the change effective.

# run hello-world command without sudo prefix to test
docker run hello-world
```
install docker compose

`sudo apt install docker-compose`

Check the docker-compose installation.

`docker-compose --version`

## Golang

Download go with wget, or manually

[download go](https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz)

Extract it into /usr/local, creating a Go tree in /usr/local/go

```
# replace go$version.$os-$arch.tar.gz with the actual go package name
tar -C /usr/local -xzf go$version.$os-$arch.tar.gz
```

Add go binary into PATH environment variable by adding a line into *.profile*
in $HOME/.profile, so that you can use *go* command everywhere

`export PATH=$PATH:/usr/local/go/bin`

The change may not work until you restart the terminal session, or you can

`source $HOME/.profile`

Check Go configuration by `go env`, and version by `go version`. Make sure the version is 1.14.x

Set gopath to point your go workspace containing fabric code.
Fabric building framework requires that the gopath must be set and contain only a single directory. 
Adding a line into your ~/.bashrc (if your are using bash as your terminal). 

`export GOPATH=$HOME/go`

Extend your command search path to include the Go bin directory.

`export PATH=$PATH:$GOPATH/bin`

## node.js and python are optional

With node.js and python2.7, you can develop applications with fabric node.js sdk.

# Prepare Fabric Sample

## Automatically execute

`curl -sSL https://bit.ly/2ysbOFE | bash -s`

## Manually install sample
If the command above return connection errors(it may happen), you have to download the scripts and run it manually.

Enter $GOPATH and download the script into $gopath/script/bootstrap.sh from
[bootstrap.sh](https://github.com/hyperledger/fabric/blob/master/scripts/bootstrap.sh)

Change file mode to executable,

`chmod +x ./scripts/bootstrap.sh`

and run the script.

`./script/bootstrap.sh`

This script does following things
1. clone fabric-sample repo, and checkout v2.1.0
2. download binaries and config files
3. download docker images

After the program execution completed, check your *bin* and *config* directory in current workspace. Make sure these two directories are inside *fabric-sample* directory.

There will be several binaries. You could add the *bin* path into PATH environment variable for convenience.
- configtxgen
- configtxlator
- cryptogen
- discover
- fabric-ca-client
- fabric-ca-server
- idemixgen
- orderer
- peer

# Use Fabric Test Network

## Bring up the testnet

Navigate to ***./fabric-sample/test-network***, find the script ***network.sh***, and check functions of the script.

```
cd ./fabric-sample/test-network
./network.sh -h
```

Clean previous containers and start a testnet. Every time before running a new testnet, you should clean previous containers.

```
./network.sh down
./network.sh up
```

This will create a fabric test network with 2 peer nodes and 1 orderer node. You can check these processes by

`docker ps -a`

There should be 3 docker images running. The testnet has 2 consortium members, Org1 and Org2. The orderer node belongs to an organization that provides ordering service of the network.

## Create channel

Now you can create a channel in the testnet between Org1 and Org2.

`./network.sh createChannel`

With the flag *-c* you could specify a customized channel name, channel1 for example.

`./network.sh createChannel -c channel1`

The flag *-c* can also be used to create another channel by givin it a different name.

## Chaincode on the channel

After a channel was created, you can install chaincode(smart contract) into peer and deploy it on the channel.

`./network.sh deployCC`

This will install a chaincode fabcar, a car info and trade management smart contract, into both peers, and deploy it into the channel specified, mychannel by default. The source code can be found in ***./fabric-sample/chaincoe/fabcar***

## Interact with the network

After smart contract deployed, you can interact with your network with ***peer*** CLI, located in ***/fabric-sample/bin***.

You need to operate in ***test-network*** directory.
Add binaries into your ***PATH***, and set your ***FABRIC_CFG_PATH*** to point to *core.yaml* file in the sample repo. Then you can set environment variables to allow you to operate ***peer*** CLI as Org1.

```
export PATH=${PWD}/../bin:${PWD}:$PATH

export FABRIC_CFG_PATH=$PWD/../config/

# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

Now you can use ***peer*** to interact with smart contract fabcar.

**Query**. Get the list of all cars.

`peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'`

**Invoke**. Change the owner of a car.

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls true --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"changeCarOwner","Args":["CAR9","Dave"]}'
```

You can know more about interfaces and parameters of fabcar chaincode in source code file memtioned above.

You can confirm the change worked by query agian. This time we use Org2.

Set environment variables for Org2 first.

```
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

And then query the info of car9.

`peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryCar","CAR9"]}'`

The owner of car9 should be changed to Dave.

## Bring down the network

The command will stop and remove the node and chaincode containers, delete the organization crypto material, and remove the chaincode images from your Docker Registry. The command also removes the channel artifacts and docker volumes from previous runs.

`./network.sh down`
