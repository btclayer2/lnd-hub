## 安装[Docker](https://www.docker.com/)

- 打开[Docker官网](https://www.docker.com/)，下载对应系统版本

  ![image-20241128wGu4uZRu@2x](../assets/images/image-20241128wGu4uZRu@2x.png)

  下载后安装,之后打开Docker Desktop，等待Docker 正常运行。

- 使用命令行安装

  - Window

    ```powershell
    winget install Docker.DockerDesktop
    ```

  - Macos

    ```shell
    brew install docker --cask
    ```

    或者

    ```shell
    brew install orbstack --cask
    ```

    

  - Linux

    ```shell
    curl -fsSL https://get.docker.com | bash -s docker
    sudo systemctl start docker
    sudo systemctl enable docker.service
    ```


## 配置LND

1. 选择或者创建为lnd节点存储数据的文件夹

   ```shell
   mkdir ./lnd && cd lnd
   ```

   

2. 创建配置文件`lnd.conf`，可以建议使用当前的 `lnd.conf.example`

    ```shell
    cp lnd.conf.example lnd.conf
    
    vim lnd.conf
    ```

    如果你有自己的BTC全节点的话，可以替换`bitcoin.node`设置为 `btcd` 或者 `bitcoind`

    需要对应的修改`[neutrino]` 

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

    **注意**：如果替换`bitcoin.node` 为全节点 `btcd` 或者 `bitcoind` 且未设置远程节点信息，lnd将在内部启动一个全节点同步，这将会花费很长的时间和占用较大的存储盘资源

    

    如果没有全节点，可以使用`lnd.conf.example`中设置的`neutrino` 轻节点，lnd会自动同步数据（大概10分钟）

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

## 运行LND

```shell
docker run --name lnd --rm -d --network host -v ./lnd:/root/.lnd lightninglabs/lnd:v0.18.3-beta
```

此时未创建钱包，如果使用了`bitcoin.node=neutrino` 配置，lnd节点会开始运行`neutrino` 轻节点，并开始同步数据，同步数据需要一点时间，同步期间无法使用钱包功能

## 创建钱包

```shell
docker exec -it lnd lncli create
```

根据提示输入密码(输入密码不会显示)

```shell
Input wallet password:
Confirm password:
```

如果已有钱包可以根据`y`或者`x`导入钱包，如果没有钱包可以输入`n`创建新钱包

``` shell
Do you have an existing cipher seed mnemonic or extended master root key you want to use?
Enter 'y' to use an existing cipher seed mnemonic, 'x' to use an extended master root key
or 'n' to create a new seed (Enter y/x/n):
```

一般助记词部分的密码是空，直接回车

```shell
Your cipher seed can optionally be encrypted.
Input your passphrase if you wish to encrypt it (or press enter to proceed without a cipher seed passphrase):
Confirm password:
```

如果是创建新钱包，记得在此步骤记住当前助记词

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



## 完成

到此节点创建和运行已经完成，认证使用的admin.macaroon在当前文件夹的`./lnd/data/chain/bitcoin/{network}/admin.macaroon`