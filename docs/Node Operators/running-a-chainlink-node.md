---
layout: nodes.liquid
date: Last Modified
title: "Running a Chainlink Node"
permalink: "docs/running-a-chainlink-node/"
whatsnext: {"Fulfilling Requests":"/docs/fulfilling-requests/", "Performing System Maintenance":"/docs/performing-system-maintenance/", "Miscellaneous":"/docs/miscellaneous/", "Best Security and Operating Practices":"/docs/best-security-practices/"}
metadata:
  title: "Running a Chainlink Node"
  description: "Run your own Chainlink node using this guide which explains the requirements and basics for getting started."
  image:
    0: "/files/OpenGraph_V3.png"
---
In this section, we'll explain the requirements and basics for running your own Chainlink node.

It's important to note that nodes can fulfill requests for open APIs out-of-the-box using our core [Adapters](../core-adapters/) without needing any additional configuration.

If you would like to provide data from an authenticated API, you can add an [external adapter](../external-adapters/) to enable connectivity through the Chainlink node.

Hardware requirements are light. The only heavy part is you'll need a blockchain node connection. If you use a 3rd party (defined below), you can use a machine with as little as 10GB of storage and 2GB of RAM.

![node-diagram](/files/ab5762f-end-to-end-diagram.png)

# Running From Source

To run a Chainlink node from source, use the [following instructions](https://github.com/smartcontractkit/chainlink#install).

# Using Docker

It's recommended to run the Chainlink node with <a href="https://www.docker.com/" target="_blank" rel="noreferrer, noopener">Docker</a>. This is because we continuously build and deploy the code from our <a href="https://github.com/smartcontractkit/chainlink" target="_blank" rel="noreferrer, noopener">repository on Github</a>, which means you don't need a complete development environment to run a node.

## Requirements

- <a href="https://docs.docker.com/install/" target="_blank" rel="noreferrer, noopener">Docker-CE</a>. Quick instructions for setting up Docker is below:

```shell Amazon Linux 2
sudo amazon-linux-extras install -y docker
sudo systemctl start docker
sudo gpasswd -a $USER docker
exit
# log in again
```
```shell CentOS
curl -sSL https://get.docker.com/ | sh
sudo systemctl start docker
sudo usermod -aG docker $USER
exit
# log in again
```
```shell Debian
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker $USER
exit
# log in again
```
```shell Fedora
curl -sSL https://get.docker.com/ | sh
sudo systemctl start docker
sudo usermod -aG docker $USER
exit
# log in again
```
```shell Ubuntu
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker $USER
exit
# log in again
```

- A fully synced Ethereum client with websockets enabled. Client specific instructions can be found below:
  - [Run Geth](../run-an-ethereum-client/#geth)
  - [Run OpenEthereum](../run-an-ethereum-client/#parity)
  - [Use an external service](../run-an-ethereum-client/#external-services)
### Create a directory

Once you have your Ethereum client running and fully synced, you're ready to run the Chainlink node.

Create a local directory to hold the Chainlink data:

```shell Rinkeby
mkdir ~/.chainlink-rinkeby
```
```shell Kovan
mkdir ~/.chainlink-kovan
```
```shell Mainnet
mkdir ~/.chainlink
```

> **_Other Supported Networks:_**  Chainlink is blockchain agnostic technology. The [LINK Token Contracts](../link-token-contracts/) page details networks which support the LINK token. You can setup your node to provide data to any of these blockchains.

### Create an Environment File

Run the following as a command to create an environment file and populate with variables specific to the network you're running on. For a full list of available configuration variables, click [here](../configuration-variables/).

```shell Rinkeby
echo \"ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=4
MIN_OUTGOING_CONFIRMATIONS=2
LINK_CONTRACT_ADDRESS=0x01BE23585060835E02B77ef475b0Cc51aA1e0709
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
GAS_UPDATER_ENABLED=true
ALLOW_ORIGINS=*\" > ~/.chainlink-rinkeby/.env
```
```shell Kovan
echo \"ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=42
MIN_OUTGOING_CONFIRMATIONS=2
LINK_CONTRACT_ADDRESS=0xa36085F69e2889c224210F603D836748e7dC0088
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
GAS_UPDATER_ENABLED=true
ALLOW_ORIGINS=*\" > ~/.chainlink-kovan/.env
```
```shell Mainnet
echo \"ROOT=/chainlink
LOG_LEVEL=debug
ETH_CHAIN_ID=1
CHAINLINK_TLS_PORT=0
SECURE_COOKIES=false
GAS_UPDATER_ENABLED=true
ALLOW_ORIGINS=*\" > ~/.chainlink/.env
```

### Set your Ethereum Client URL

[block:callout]
{
  "type": "warning",
  "body": "If you're using a 3rd party service to connect to the blockchain, skip to the [External Provider](#ethereum-client-as-an-external-provider) section to set the `ETH_URL` environment variable. We provide general guidance, but you will need to obtain the websocket connection string to add to your environment file.",
  "title": "Using an external Ethereum client?"
}
[/block]

### Ethereum Client on the Same Machine

Next you need to get the URL for the Ethereum client. The command below will help you obtain the IP address of the container that your Ethereum client is running on. **This will only work if you have started an Ethereum client on the same machine as your Chainlink node.**

```shell Local
ETH_CONTAINER_IP=$(docker inspect --format '{{ .NetworkSettings.IPAddress }}' $(docker ps -f name=eth -q))
```

Then run the following command to add the Ethereum client's URL to your environment file. If you are using an external Ethereum client, use the External tab below, and update `$ETH_CONTAINER_IP` to the websocket address used for connectivity.

```shell Rinkeby
echo \"ETH_URL=ws://$ETH_CONTAINER_IP:8546\" >> ~/.chainlink-rinkeby/.env
```
```shell Kovan
echo \"ETH_URL=ws://$ETH_CONTAINER_IP:8546\" >> ~/.chainlink-kovan/.env
```
```shell Mainnet
echo \"ETH_URL=ws://$ETH_CONTAINER_IP:8546\" >> ~/.chainlink/.env
```

### Ethereum Client as an External Provider

If you are using an external provider for connectivity to the Ethereum blockchain or you are running an Ethereum client on a separate instance, you may use the command below for your network. Be sure to update the value for `CHANGEME` to the value given by your provider or the address and port of your separate instance.

```shell Rinkeby
echo \"ETH_URL=CHANGEME\" >> ~/.chainlink-rinkeby/.env
```
```shell Kovan
echo \"ETH_URL=CHANGEME\" >> ~/.chainlink-kovan/.env
```
```shell Mainnet
echo \"ETH_URL=CHANGEME\" >> ~/.chainlink/.env
```

[block:callout]
{
  "type": "warning",
  "body": "Ganache is a mock testnet and it doesn't work with Chainlink because of that. To use the features of the network, you need to deploy your contract on a real environment: one of the testnets or mainnets. The full list of supported environments can be found [here](../link-token-contracts/).",
  "title": "Running Chainlink Node on Ganache"
}
[/block]

### Set the Remote DATABASE_URL Config

You will need to connect your Chainlink node with a remote PostgreSQL database. See the [Connecting to a Remote Database](../connecting-to-a-remote-database/) page for more information. Use the example below to configure your `DATABASE_URL` setting in your environment file, replacing `$VARIABLES` with their actual values.

- `$USERNAME`: The database username (must be owner)
- `$PASSWORD`: The user's password
- `$SERVER`: The server name or IP address of the database server
- `$PORT`: The port that the database is listening on
- `$DATABASE`: The database to use for the Chainlink node (i.e. "postgres")

[block:callout]
{
  "type": "warning",
  "body": "If you're testing you can add `?sslmode=disable` to the end of your `DATABASE_URL`. However you should *never* do this on a production node."
}
[/block]

```shell Rinkeby
echo \"DATABASE_URL=postgresql://$USERNAME:$PASSWORD@$SERVER:$PORT/$DATABASE\" >> ~/.chainlink-rinkeby/.env
```
```shell Kovan
echo \"DATABASE_URL=postgresql://$USERNAME:$PASSWORD@$SERVER:$PORT/$DATABASE\" >> ~/.chainlink-kovan/.env
```
```shell Mainnet
echo \"DATABASE_URL=postgresql://$USERNAME:$PASSWORD@$SERVER:$PORT/$DATABASE\" >> ~/.chainlink/.env
```

For a primary/secondary Chainlink node architecture, you may also want to set the `DATABASE_TIMEOUT` configuration as well. Setting `DATABASE_TIMEOUT` to 0 allows a secondary node to wait for the lock to be released on the database indefinitely.

```shell Rinkeby
echo \"DATABASE_TIMEOUT=0\" >> ~/.chainlink-rinkeby/.env
```
```shell Kovan
echo \"DATABASE_TIMEOUT=0\" >> ~/.chainlink-kovan/.env
```
```shell Mainnet
echo \"DATABASE_TIMEOUT=0\" >> ~/.chainlink/.env
```

### Start the Chainlink Node

Now you can run the Docker image. Replace `<version>` with your desired version. Tag versions are available in the [Chainlink docker hub](https://hub.docker.com/r/smartcontract/chainlink/tags). *The `latest` version does not work.*

```shell Rinkeby
cd ~/.chainlink-rinkeby && docker run -p 6688:6688 -v ~/.chainlink-rinkeby:/chainlink -it --env-file=.env smartcontract/chainlink:<version> local n
```
```shell Kovan
cd ~/.chainlink-kovan && docker run -p 6688:6688 -v ~/.chainlink-kovan:/chainlink -it --env-file=.env smartcontract/chainlink:<version> local n
```
```shell Mainnet
cd ~/.chainlink && docker run -p 6688:6688 -v ~/.chainlink:/chainlink -it --env-file=.env smartcontract/chainlink:<version> local n
```

[block:callout]
{
  "type": "info",
  "title": "Local Database",
  "body": "If you're running a local database you may need to add `--network host` to the end of the command above."
}
[/block]

The first time running the image, it will ask you for a password and confirmation. This will be your wallet password that you can use to unlock the keystore file generated for you. Then, you'll be prompted to enter an API Email and Password. This will be used to expose the API for the GUI interface, and will be used every time you log into your node. When running the node again, you can supply the `-p` option with a path to a text file containing the wallet key password, and a `-a` option, pointing to a text file containing the API email and password. Instructions on how to do that are [here](../miscellaneous/#use-password-and-api-files-on-startup).

[block:callout]
{
  "type": "info",
  "body": "You will need to send some ETH to your node's address in order for it to fulfill requests. You can view your node's ETH address when the node starts up or on the Configuration page of the GUI."
}
[/block]

You can now connect to your Chainlink node's UI interface by navigating to <a href="http://localhost:6688" target="_blank" rel="noreferrer, noopener">http://localhost:6688</a>. If using a VPS, you can create a <a href="https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/" target="_blank" rel="noreferrer, noopener">SSH tunnel</a> to your node for `6688:localhost:6688` to enable connectivity to the GUI. Typically this is done with `ssh -i $KEY $USER@$REMOTE-IP -L 6688:localhost:6688 -N`. A SSH tunnel is recommended over opening up ports specific to the Chainlink node to be public facing. See our [Best Security and Operating Practices](../best-security-practices/) page for more details on how to secure your node.
