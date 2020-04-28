# install prerequisites
## git, curl (take git as an example. for curl, just replace *git* with *curl*)

in ubuntu, install git with apt

`sudo apt install git`

check git installation

`git --version`

## docker and docker compose

install docker engine with the command

`sudo apt install docker.io`

docker.io is the name of deb package of docker engine for ubuntu

check docker version, and make sure it is greater than 17.06.2

`docker --version`

keep docker running with systemd

`sudo systemctl start docker`

automatically start docker daemon when system starts

`sudo systemctl enable docker`

(optional) add user to docker group to manage docker with non-root user

```
# create a group named docker
sudo groupadd docker

# add your user into the docker group
sudo usermod -aG docker $USER

# activate the change
newgrp docker
# if this does not work, try to end current session and start a new one, or
# if work in vm, try to restart the machine to make the change effective.

# run hello-world command without sudo prefix to test
docker run hello-world
```
install docker compose

`sudo apt install docker-compose`

check the docker-compose installation

`docker-compose --version`

## golang

download go with wget, or manually

[download go](https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz)

extract it into /usr/local, creating a Go tree in /usr/local/go

```
# replace go$version.$os-$arch.tar.gz with the actual go package name
tar -C /usr/local -xzf go$version.$os-$arch.tar.gz
```

add go binary into PATH environment variable by adding a line into *.profile*
in $HOME/.profile, so that you can use *go* command everywhere

`export PATH=$PATH:/usr/local/go/bin`

the change may not work until you restart the terminal session, or you can

`source $HOME/.profile`

check Go configuration by `go env`, and version by `go version`. make sure the version is 1.14.x

set gopath to point your go workspace containing fabric code.
fabric building framework requires that the gopath must be set and contain only a single directory. 
adding a line into your ~/.bashrc (if your are using bash as your terminal). 

`export GOPATH=$HOME/go`

extend your command search path to include the Go bin directory

`export PATH=$PATH:$GOPATH/bin`

## node.js and python are optional

with node.js and python2.7, you can develop applications with fabric node.js sdk.

# fabric sample

## automatical execution

`curl -sSL https://bit.ly/2ysbOFE | bash -s`

## manually install sample
if the command above return connection errors(it may happen), you have to download the scripts and run it manually.

enter $GOPATH and download the script into $gopath/script/bootstrap.sh from
[bootstrap.sh](https://github.com/hyperledger/fabric/blob/master/scripts/bootstrap.sh)

change file mode to executable

`chmod +x ./scripts/bootstrap.sh`

and run the script

`./script/bootstrap.sh`

after the program execution completed,check your *bin* directory in current workspace. 

there will be several binaries. you could add the *bin* path into PATH environment variable for convenience.
- configtxgen
- configtxlator
- cryptogen
- discover
- fabric-ca-client
- fabric-ca-server
- idemixgen
- orderer
- peer






