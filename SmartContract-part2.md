## What happened last time ...
  In the previous section of series - [Getting Ready for Smart Contract Development on EOS](), we took a deep dive in setting up the correct enviornment needed for interacting with a blockchain infrastructure managed by the EOS.io software. We learnt how to spin up an developer sandbox environment through pulling official pre-built development images through docker. We also learnt how to properly set up a single blockchain, create the neccessary accounts, keys, and wallets in order to deploy contracts in to the blockchain. 

## In this article ...
  In this section part 2 section of the series we'll continue to explore how accounts and wallets interact with smart contracts that are deployed on the blockchain. We'll first get a taste of smart contract development by simply constructing a "Hello World" contract and deploying it under the sandbox environment which we have created in the previous part of the tutorial.

  explore how to deploy a simple token contract on the eos blockchain, then we will proceed to issue the tokens and calling functions defined within the smart contract

## Before we start ...
  Following the wallet set up that we have done in the last article in [Getting Ready for Smart Contract Development on EOS](), we should first get more clearer sense of how wallet, keys and accounts interact within the eos ecosystem. Cleos - which is the commandline tool that acts as an interface between nodeos - the actual node daemon hosting the blockchain and keosd - the component responsible for managing keys in the wallet. The eos blockchain hosted by nodeos will only recognize accounts as a identity that's associated with each individual entity acting and ttransacting on the blockchain, and these accounts are connected by the crypotgraphic identity of private key that we are accustomed to in other blockchain platform. These private keys acts as the crediential to prove to the eos blockchain that an individual is indeed the owner of the account. The wallet itself, however, doesn't directed interface with the eos blockchain, it's only responsibility is to manage and keep the private key from being exposed. In the sandbox developer environment that we have created in part 1 to prepare ourself for smart contract development, the sandbox eos blockchain will actually generate a block producing account called eosio. The specific private key of this eosio account can be found within the config.ini file. Without importing the private key to anyone of our active wallet, we won't be able to gain access to the eosio account. Since in eos, account generation can only be done by an existing account, we won't be able to genearte additional accounts, signing transaction and deploying smart contracts; thus, we have imported the key in one of our generated wallet of "producer", in practice, the key can be imported into any of the active wallet or even multiple wallets. 

## Generating the Hello World Contract.

  Let's first create a "hello world" contract by creating a folder call "hello" and change the directory into the folder,

  ```bash
  mkdir hello && cd hello
  ```
  For the simplicity of this tutorial, we shall create a pre-written smart contract that the eos developer tutorial has kindly porvided,

```c++
#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>
using namespace eosio;

class hello : public eosio::contract {
  public:
      using contract::contract;

      /// @abi action 
      void hi( account_name user ) {
         print( "Hello, ", name{user} );
      }
};

EOSIO_ABI( hello, (hi) )
```
We will dive deeply in terms of writing our own smart contract in our future tutorials. The "hello world" smart contract encompasses a simple function which upon calling with a parameter that defines the an account name through a signed transaction, will return the string "Hello, " and the account name.

For the next step, we need to compile the hello.cpp into a WebAssembly(WASM) file .wast and abi code that is neccessary for the deployment of the smart contract through a cli tool called eosiocpp that is packaged into the eos dev docker image. In order to use eosiocpp, we need to access the one of docker container that we have created and execute the commands from with in.

First, place the "hello" folder which we just have created into "keosd-data-volume" - the shared folder mountpoint between the docker container keosd and the host system. Note, since the folder itself is created by docker we will be needing super user privilege when moving files into it.

```bash
sudo mv hello keosd-data-volume/
```

Get into the bash of the keosd container by,
```bash
docker container exec -it keosd bash
```

Change directory to where the "hello" folder is located within the container.
```bash
cd /opt/eosio/bin/data-dir/hello
```
*Note that /opt/eosio/bin/data-dir path is actually configured in the yaml file that brought up the containers.*

Generate web assembly and abi through the compiler.
```bash
eosiocpp -o hello.wast hello.cpp &&
eosiocpp -g hello.abi hello.cpp
```
*Note there may be warnings about no ricardian clauses found which can be safely ignore*

## Deploying the Hello World Contract.

Let's create an contract account called "hello.code" to deploy smart contract. Creating a separate account for the purpose of deployment is preferred since other participants will access the contract that we've created through the reference of the contract account. 

```bash
cleos create account eosio hello.code ${owner public key} ${active public key}
```
*Note the corresponding private key to the owner public key and active public key needs to be in an unlocked wallet in order to sign of transaction with account that we have created.*

The owner public key and active public defines the permission level that the account is associated with. By default each account will have two native named permission which is `owner` and `active`. The `owner` permission level have the greatest authority of the account and will be used in cases where greatest authority is required such as changing the ownership of the account. The `active` permission on the other hand is more commonly used for signing transactions associated with the account. Custom permission level can also be implmented to extend controls of the account. 

Let's deploy our "hello world" contract and sign off the transaction with the `active` permission of our newly generated contract account `hello.code`.

```bash
# if we are inside the docker container under the /opt/eosio/bin/data-dir path
cleos -u http://nodeosd:8888 set contract hello.code ./hello -p hello.code@active

# if we are in the host environment
cleos set contract hello.code /opt/eosio/bin/data-dir path/hello -p hello.code@active
```
*Note since we are deploying our contract with in the `keosd` docker container environment, we need to specifiy the location that our nodeos is running on within the network with `-u http://nodeosd:8888`.*

After our contact has been successfully deployed, let's try to invoke the contract with our `user` account. Call the `hi` function speicified in the smart contract by referrin to the contract account `hello.code`

```
cleos -u http://nodeosd:8888 push action hello.code hi '["user"]' -p user@active
executed transaction: 4c10c1426c16b1656e802f3302677594731b380b18a44851d38e8b5275072857  244 bytes  1000 cycles
#    hello.code <= hello.code::hi               {"user":"user"}
```

If you have attach the to log output of the nodeos docker container, you should see output similar to the following,
```bash
1025500ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00004de945a23f63... #19945 @ 2018-05-25T19:17:05.500 signed by eosio [trxs: 0, lib: 19944, confirmed: 0]
1025830ms thread-0   apply_context.cpp:28          print_debug          ] 
[(hello.code,hi)->hello.code]: CONSOLE OUTPUT BEGIN =====================
Hello, user
[(hello.code,hi)->hello.code]: CONSOLE OUTPUT END   =====================
1026000ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00004deaebee2dc5... #19946 @ 2018-05-25T19:17:06.000 signed by eosio [trxs: 1, lib: 19945, confirmed: 0]
```

## Deploying a token contract

  Now that we have successfully deployed our first "hello world" contract, let's try to deploy another token contract that can be used to issuing your own tokens on the eos platform.

  Again, let's start by creating a token contract account that's responsible for managing the token contract.

```bash
cleos create account eosio eosio.token ${owner public key} ${active public key}
```
  We proceed to deploy the pre-compiled contract `eosio.token` found under the `/contracts` folder within the keosd docker container

  We can peek at the functions that are defined within the contract by looking at the `.hpp` file.
  `contracts/eosio.token/eosio.token.hpp:`
  ```
    void create( account_name issuer,
                asset        maximum_supply );

    void issue( account_name to, asset quantity, string memo );

    void transfer( account_name from,
                  account_name to,
                  asset        quantity,
                  string       memo );
  ```

  The concise way to call the `create` function to create a new token:
  
  ```bash
cleos push action eosio.token create '[ "eosio", "1000000000.0000 SYS"]' \
         -p eosio.token@active
executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

  ```

  The more verbose way:
  ```bash
cleos push action eosio.token create '[ "eosio", "1000000000.0000 SYS"]' \
         -p eosio.token@active
executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}
  ```

  Either way the command have created a new token SYS with a precision of 4 decimals and a maximum supply of 1000000000.0000 SYS.

  ### Issue Tokens and Transfering Them.
  
  The following command will issue 100.0000 SYS to the account `user`:
```bash
cleos push action eosio.token issue '[ "user", "100.0000 SYS", "memo" ]' \
        -p eosio@active
```

The output should look similar as the following indicating the successful transaction.

```bash
executed transaction: 822a607a9196112831ecc2dc14ffb1722634f1749f3ac18b73ffacd41160b019  268 bytes  1000 cycles
#   eosio.token <= eosio.token::issue           {"to":"user","quantity":"100.0000 SYS","memo":"memo"}
>> issue
#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
>> transfer
#         eosio <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
#          user <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
```

Since now, the `user` account have some amount of SYS token, the `user` account can invoke the `transfer` function to transfer some tokens to another account such as `test`:

```bash
cleos push action eosio.token transfer \
        '[ "user", "tester", "25.0000 SYS", "m" ]' -p user@active
```
  Notice that this time the transaction is actually signed by the user account ranther than the token contract account.

  The output should look similar as the following indicating the successful transaction.
  
```bash
executed transaction: 06d0a99652c11637230d08a207520bf38066b8817ef7cafaab2f0344aafd7018  268 bytes  1000 cycles
#   eosio.token <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
>> transfer
#          user <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
#        tester <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
```

## Moving Forward

  Now that we have learnt how to deploy and interact with smart contracts, it's essential to start defining our own contract logic that we see fit for the use cases of our Dapp. In the next part of our series, we'll dive deeper into the details of how to exactly write smart contract. Stay tuned ...The output should look something as the following indicating the successful transaction.