# Introduction

EOS being a newborn in blockchain space, promises to deliver vertical and horizontal scaling of decentralized applications unmatched by the other Blockchain platform out there. The technology and infrastructure of the EOS blockchain will eventually have the capability of driving millions of transactions per second through a fast DPos - Delegated Proof of Stake - consensus algorithm. At EOS9CAT, we are developing applications with the amazing technology. We feel that it's absolutely crucial to share our experience and hope to inspire you to build something of your own. 

# Setting Up

The EOS developement ecosystem currently features three main toolset to interact the blockchain:
* **Nodeos** is a daemon that is used to host and run EOS node.
* **Keosd** is a light-weight wallet client responsible for protecting keys and signing transactions with them. 
* **Cleos** is a command line tool to interact with EOS blockchain and serves as bridging component between Nodeos and Keosd.


## Step One: Building EOS

Make sure your system has 8GB of RAM and 20GB of free hardware space.


#### Install Docker

Docker is a container management service that runs on top of the os kernel to allow for easy applicaiton development and deployment by allowing developers to synthesis a sandbox environment with fine-tuned system environments, variables, and packages.

In order for the to develop with docker containers, it's neccessary to install the necessary files and programs, please follow this guide: https://docs.docker.com/install/ and install docker for your corresponding os.

### Option 1(One Node Configuration with Docker)

#### Pulling the Dev Docker Image

EOSIO provides a Docker image on docker hub - which is the equivalent of Github for Docker - that is pre-packaged with the neccessary toolset needed for developing smart contracts for the EOS blockchain.

```
docker pull eosio/eos-dev
```

*Note: the eos-dev image is significantly

#### Start Nodeos

In order to initiate a EOS blockchain startup that runs with a single node, type in the command:

```
sudo docker container run --rm --name eosio -d -p 8888:8888 -p 9876:9876 -v /tmp/work:/work -v /tmp/eosio/data:/mnt/dev/data -v /tmp/eosio/config:/mnt/dev/config eosio/eos-dev  /bin/bash -c "nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::wallet_plugin --plugin eosio::producer_plugin --plugin eosio::history_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin --plugin eosio::http_plugin -d /mnt/dev/data --config-dir /mnt/dev/config --http-server-address=0.0.0.0:8888 --access-control-allow-origin=* --contracts-console"
```

To spin a docker container from our eos-dev image that was pulled earlier and create an ideal EOS development environment, it's crucial apply certain flags that'll aid through the whold developement process.
* **rm** flag allows the container to be automatically removed once stopped.
* **d** flag allows allows container to be run in detached mode (as a daemon in the background)
* **p** flag allows port binding between host and container - which is useful to access API interface exposed within the container as we'll see later.
* **v** flag allows for folder mounting between the host and the storage within the container. 

run the following command to check if the node is working properly:
```
sudo docker logs --tail 10 eosio
```

The output will be somthing similar to this which is logging info that you'll get when you run nodeos in a native environment:

```
1929001ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366974ce4e2a... #13929 @ 2018-05-23T16:32:09.000 signed by eosio [trxs: 0, lib: 13928, confirmed: 0]
1929502ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366aea085023... #13930 @ 2018-05-23T16:32:09.500 signed by eosio [trxs: 0, lib: 13929, confirmed: 0]
1930002ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366b7f074fdd... #13931 @ 2018-05-23T16:32:10.000 signed by eosio [trxs: 0, lib: 13930, confirmed: 0]
1930501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366cd8222adb... #13932 @ 2018-05-23T16:32:10.500 signed by eosio [trxs: 0, lib: 13931, confirmed: 0]
1931002ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366d5c1ec38d... #13933 @ 2018-05-23T16:32:11.000 signed by eosio [trxs: 0, lib: 13932, confirmed: 0]
1931501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366e45c1f235... #13934 @ 2018-05-23T16:32:11.500 signed by eosio [trxs: 0, lib: 13933, confirmed: 0]
1932001ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000366f98adb324... #13935 @ 2018-05-23T16:32:12.000 signed by eosio [trxs: 0, lib: 13934, confirmed: 0]
1932501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00003670a0f01daa... #13936 @ 2018-05-23T16:32:12.500 signed by eosio [trxs: 0, lib: 13935, confirmed: 0]
1933001ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 00003671e8b36e1e... #13937 @ 2018-05-23T16:32:13.000 signed by eosio [trxs: 0, lib: 13936, confirmed: 0]
1933501ms thread-0   producer_plugin.cpp:585       block_production_loo ] Produced block 0000367257fe1623... #13938 @ 2018-05-23T16:32:13.500 signed by eosio [trxs: 0, lib: 13937, confirmed: 0]
```

If you would like to stream the logs, you can attach back to the container with the command:

```
sudo docker container attach eosio
```

or simply:

```
sudo docker logs -f
```

Finally, check the this address in the host browser to ensure that the RPC interface and port binding is working as expected:

```
http://localhost:8888/v1/chain/get_info
```

The browser should return a json object that should be similar to this:

```
{
  "server_version": "5875549c",
  "chain_id": "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
  "head_block_num": 13764,
  "last_irreversible_block_num": 13763,
  "last_irreversible_block_id": "000035c3a6a03d75ce5b42952dce1636d8f7c46254170dd10f569f532b529581",
  "head_block_id": "000035c479992b3350af6a87e1b7f2e34237dbe53ed873355262a4ac5b585c5b",
  "head_block_time": "2018-07-05T01:03:07.000",
  "head_block_producer": "eosio",
  "virtual_block_cpu_limit": 200000000,
  "virtual_block_net_limit": 1048576000,
  "block_cpu_limit": 199900,
  "block_net_limit": 1048576
}
```

#### Using Cleos

Cleos being the command line interface that interacts with the EOS blockchain through nodeos and manage the wallects through the Keosd software, will spawn an Keosd on first launch. Since direct interaction with the blockchain will be initiated through the command line most of the time, it'll be more convenient to create a bash alias in the host environment to quickly interact with sandbox environment laying within the container.

```
alias cleos='docker exec eosio /opt/eosio/bin/cleos --wallet-url http://localhost:8900'
```

The **wallet-url** flag should point to the address of that Keosd is running on. 

*Note: by default when spwaning Keosd through Cleos, the address of Keosd will be set to port 8900 in order to avoid conflict with Nodeos running at port 8888. If Keosd is directly ran without any arguments, it will use the same http-server-address specified in the config.ini shared with Nodeos, which will result in Keosd set to port 8888.


Run cleos command with a help flag in host terminal to verify the alias:
```
cleos --help
```

*Note: the arguments passed with the alias will refer to the environment variable and path that's associated with container context instead of the host.*

Now we are ready for the next step.



### Option 2(Build with Docker Compose)

Create a file called docker-compose.yaml with the following content:

```
version: "3"

services:
  nodeosd:
    container_name: nodeosd
    image: eosio/eos-dev:latest
    command: /opt/eosio/bin/nodeosd.sh --data-dir /opt/eosio/bin/data-dir -e
    hostname: nodeosd
    ports:
      - 8888:8888
      - 9876:9876
    expose:
      - "8888"
    volumes:
      - nodeos-data-volume:/opt/eosio/bin/data-dir

  keosd:
    container_name: keosd
    image: eosio/eos-dev:latest
    command: /opt/eosio/bin/keosd --wallet-dir /opt/eosio/bin/data-dir --http-server-address=127.0.0.1:8900
    hostname: keosd
    links:
      - nodeosd
    volumes:
      - keosd-data-volume:/opt/eosio/bin/data-dir

volumes:
  nodeos-data-volume:
  keosd-data-volume:
```

#### Start Nodeos and Keosd

Run command:
```
docker-compose up -d
```

The above command will essentially spawn two containers with one of them running the Nodeos client responsible for interacting with the blockchain and Keosd for managing the wallet.

#### Set alias

Run following command to set-alias:

```
alias cleos='docker-compose exec keosd /opt/eosio/bin/cleos -u http://nodeosd:8888 --wallet-url http://localhost:8900'
```

Since the cleos alias is set to execute process in the keosd container environment, the **wallet-url** flag is set to localhost of port 8900, and the **u** flag is set to point to the address where nodeos is running, which in this instance is another container. 

Verify the alias set correctly.


## Step Two: Create Accounts

In order to create accounts, you will need two pairs of keys: owner key pair and active key pair. Each account has a owner key pair and an active key pair. Active key pair is for signing transactions and owner key pair is responsible for the account ownership.

### Create Keys:

Run the following command twice to generate two pairs of keys:

```
cleos create key
```

You should recieve the following output signaling the successful createion of private and public key pair:

```
$ cleos create key
Private key: 5JGKHd79fsS5QKDemJYqEGwpDeADooVBGFbEcahJmEgRiXFbBBQ
Public key: EOS7XnZdBp4V2yRgSUhCMXRhVGfZQcHpa828xpTiyMX4BLUQdu28B
```

### Create Wallet:

Wallet is a place where you store all of your key pairs which may or may not associate with the permission of one or more accounts.

```
cleos wallet create
```
A default wallet will generated after passing the command, you can also speicify the name of a generated wallet by passing a the **n** flag along with the name as a argument.

wallet can be unlocked and locked by invoking following commands:
```
cleos wallet lock
```
```
cleos wallet unlock --password ${passphrase}
```
By default, a wallet will be immediately unlock after its createion, and a locked wallet will require the passphrase generated during wallet creation to unlock.

The list the current available wallet, run the command:
```
cleos wallet list
```

An array of wallet name like the above will be returned:
```
[
  "default *"
]
```

The asterisk besides the wallet name indicates that the wallet is currently unlocked and available.


### Import Key Pairs into Wallet:

Now that we a wallet and couple of key pairs generated, it's time to import the keys into the wallet. Run the following command twice and each time with a private key generated before.
```
cleos wallet import ${private key}
```

### Import Authorizing Account Key:

Since the actions performed on the blockchain must be signed using the keys associated the authorizing account(producer), we can mimic this interaction by createing a separate wallet that will import the producer's private key found within the config.ini file inside the container.

```
cleos wallet create -n producer
```

By default this should be the private key associated with the authorizing account eosio:
```
cleos wallet import -n producer 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

Run the following command to ensure that all the keys are properly imported:
```
cleos wallet keys
```

### Generate Account:

This is where keys, wallet and account all tie together. In EOS blockchain, your token, smart contract all live in your account. Each account has a owner key pair and active key pair. Active key pair is for signing for transaction and own key pair is for the account ownership. Run the following command to create an account:

```
cleos create account eosio ${accountName} ${owner public key} ${active public key}
```

In the above command line, the first key will become the account owner key and the second key will become the active key.

*Note: although that the owner public key and active public key can be the same, it's highly recommended to use two different keys in production environment for security purpose.*

Since the eosio::history_api_plugin is installed, we can run the following command to query all the accounts that are associated with a public key:


Once you have the account ready, you are all set for the development environment.
```
$ cleos get accounts ${public key}
```


## Step Three: Loading the Bios Contract

Now that the wallet and accoutns are properly set up, it's time to deploy the system contract. For the purpose of the tutorial **eosio.bios** will be used, the contract can be found within the folder of /contracts/ inside the container. In the public EOS blockchain, system contract is used manage the staking and unstaking of tokens and reserve bandwidth for CPU and network activity, and memory for contracts. Again, we'll be using the eosio producer account crediential to sign off the contract deployment. 

Run command:

```
cleos set contract eosio /contracts/eosio.bios -p eosio@active
```

The similar output of the following will denote the successful deployment of the contract:

```
Reading WAST/WASM from /contracts/eosio.bios/eosio.bios.wasm...
Using already assembled WASM...
Publishing contract...
executed transaction: 7dde788c5c75a08ee0e1ebbd7865ad43861b77386372f028db4ab99f930004eb  3728 bytes  7811 us
#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001621260037f7e7f0060057f7e7e7e7e...
#         eosio <= eosio::setabi                {"account":"eosio","abi":"0e656f73696f3a3a6162692f312e30050c6163636f756e745f6e616d65046e616d650f7065...
```


Now that we have everything set up, it's time to move on to the next stage of actually writing and deploying  smart contracts in EOS ecosystem. Stay tune for EOS9CAT's next article.

