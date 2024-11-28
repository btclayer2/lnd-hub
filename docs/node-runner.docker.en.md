## Install[Docker](https://www.docker.com/)

- Open the [Docker website](https://www.docker.com/), download the version suitable for your operating system.

  ![image-20241128wGu4uZRu@2x](../assets/images/image-20241128wGu4uZRu@2x.png)

  After downloading, install Docker and open Docker Desktop. Wait until Docker is fully running.

- Install using the shell

  - Window

    ```powershell
    winget install Docker.DockerDesktop
    ```

  - Macos

    ```shell
    brew install docker --cask
    ```

    Or

    ```shell
    brew install orbstack --cask
    ```

    

  - Linux

    ```shell
    curl -fsSL https://get.docker.com | bash -s docker
    sudo systemctl start docker
    sudo systemctl enable docker.service
    ```


## LND Configuration

1.  Choose or create a folder for storing lnd node data

   ```shell
   mkdir ./lnd && cd lnd
   ```

   

   3. Create the configuration file `lnd.conf`. You can use the provided `lnd.conf.example` as a template.

      ```shell
      cp lnd.conf.example lnd.conf
      
      vim lnd.conf
      ```

      If you have your own BTC full node, you can set `bitcoin.node` to `btcd` or `bitcoind` and adjust the `[neutrino]` section accordingly.

      - btcd

        ```toml
        [btcd]
        btcd.rpchost=
        btcd.rpcuser=
        btcd.rpcpass=
        btcd.rawrpccert=
        btcd.rpccert=
        ```
      
        

      - bitcoind

        ``` toml
        [Bitcoind]
        bitcoind.rpchost=
        bitcoind.rpcuser=
        bitcoind.rpcpass=
        bitcoind.rpcpolling=true
        ```
      
      **Note**: If you set `bitcoin.node` to a full node (`btcd` or `bitcoind`) without specifying remote node information, lnd will internally start a full node synchronization. This process will consume significant time and storage space.

      

      If you don’t have a full node, you can use the lightweight neutrino configuration from the `lnd.conf.example`. lnd will automatically sync data (approximately 10 minutes).

      ```toml
      [Application Options]
      debuglevel=trace
      maxpendingchannels=10
      alias=Bevm_client_test
      no-macaroons=false
      coin-selection-strategy=largest
      rpclisten=localhost:10009
      restlisten=localhost:8080
      no-rest-tls=true
      
      [prometheus]
      prometheus.listen=[::]:8989
      
      [Bitcoin]
      bitcoin.mainnet=false
      bitcoin.testnet=false
      bitcoin.simnet=false
      bitcoin.regtest=false
      bitcoin.signet=true
      bitcoin.node=neutrino
      
      [neutrino]
      
      [protocol]
      protocol.simple-taproot-chans=true
      ```

## Run lnd

```shell
docker run --name lnd --rm -d --network host -v ./lnd:/root/.lnd lightninglabs/lnd:v0.18.3-beta
```

At this point, the wallet is not yet created. If you use `bitcoin.node=neutrino`, the lnd node will start running a lightweight neutrino node and begin syncing data. This synchronization may take some time, and wallet functionality will be unavailable during this period.

## Create a Wallet

```shell
docker exec -it lnd lncli create
```

Follow the prompts to set a wallet password (the input will not be visible):

```shell
Input wallet password:
Confirm password:
```

If you already have a wallet, you can import it by entering `y` or `x`. If not, enter `n` to create a new wallet:

``` shell
Do you have an existing cipher seed mnemonic or extended master root key you want to use?
Enter 'y' to use an existing cipher seed mnemonic, 'x' to use an extended master root key
or 'n' to create a new seed (Enter y/x/n):
```

For the cipher seed passphrase, if not needed, simply press Enter:

```shell
Your cipher seed can optionally be encrypted.
Input your passphrase if you wish to encrypt it (or press enter to proceed without a cipher seed passphrase):
Confirm password:
```

If creating a new wallet, be sure to record the generated seed phrase:

```shell
Generating fresh cipher seed...

!!!YOU MUST WRITE DOWN THIS SEED TO BE ABLE TO RESTORE THE WALLET!!!

---------------BEGIN LND CIPHER SEED---------------
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
---------------END LND CIPHER SEED-----------------

!!!YOU MUST WRITE DOWN THIS SEED TO BE ABLE TO RESTORE THE WALLET!!!

lnd successfully initialized!
```



## Completion

At this point, the lnd node setup and initialization are complete. The authentication file admin.macaroon can be found in the directory: `./lnd/data/chain/bitcoin/{network}/admin.macaroon`.



