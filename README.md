# PulseChain Testnet V3

The PulseChain Testnet is up and running. This document will guide you through connecting Metamask to the network and bootstrapping a PulseChain node of your own.

> **Disclaimer**: This is a **Testnet**, and issues may arise as the network or certain front-ends see increased load. The team will work diligently to address any issues as they come.

> **Disclaimer 2**: The state of this network may be reset on occasion, and nothing should be considered permanent after the fork block. We will communicate ahead of time when these resets are planned.

### **Version 3** 🎉

This iteration of PulseChain is an entirely new implementation and represents a hard departure from prior testnets. 

The new implementation is based on Ethereum's POS model and implements an entirely new beacon chain while forking the Ethereum execution chain and state.
> 💭 *Previous versions were based on the POSA (proof of staked authority) model.*

**This version of the PulseChain Testnet brings the following enhancements:**
-  A revamped consensus model based on Ethereum POS.
- Decreased slot time compared with Ethereum `10s vs 12s (a ~17% speedup)`.
- Validator reward burning (= more deflationary pressure).
    + 25% fee burn + additional burn to compensate for reduced slot time.
- **Multi-client support:**
    + Consensus Clients: [Prysm-Pulse](https://gitlab.com/pulsechaincom/prysm-pulse) and [Lighthouse-Pulse](https://gitlab.com/pulsechaincom/lighthouse-pulse).
    + Execution clients: [Go-Pulse](https://gitlab.com/pulsechaincom/go-pulse) and [Erigon-Pulse](https://gitlab.com/pulsechaincom/erigon-pulse).
- Sacrifice credits have been written.
- Ethereum Staker/Validator deposits have been refunded.
- The AMM bot has been executed to rebalance various automated market-makers.

## Exploring the PulseChain Testnet

Public block and beacon explorers are available at the URLs below:

- [PulseChain Block Explorer](https://scan.v3.testnet.pulsechain.com)
- [PulseChain Beacon Explorer](https://beacon.v3.testnet.pulsechain.com)

> The PulseChain team welcomes the development 3rd-party explorers!

## Connecting Metamask

Follow these instructions to manually add the new Testnet to your Metamask plugin. A button will be available in the future to do this automatically.

**1. Click the Networks dropdown and select `Add network`** ([Screenshot](images/step1.png)).

**2. At the bottom of the network list, click `Add a network manually`** ([Screenshot](images/step2.png)).

**3. Enter the following information into the form and `save`:** ([Screenshot](images/step3.png)).
- Network Name: `PulseChain Testnet-V3`
- New RPC URL: `https://rpc.v3.testnet.pulsechain.com`
- Chain ID: `942`
- Currency Symbol: `tPLS`
- Block explorer URL: `https://scan.v3.testnet.pulsechain.com`

**Congratulations**! You are now connected to the PulseChain Testnet-V3. Existing ethereum accounts that had balances as of block `16,492,699` (*Jan-26-2023 06:11:35 PM +UTC*) will have the equivalent balance on the PulseChain Testnet in addition to the **beta release** of credits from the sacrifice phase as well as any ethereum staking deposit refunds.

## Getting tPLS to use on the PulseChain Testnet

To get tPLS you can use the tPLS faucet.

1. Navigate to the tPLS faucet https://faucet.v3.testnet.pulsechain.com/
2. Connect your Metamask wallet by clicking on the button.
3. Enter the address you want to send tPLS to and click the `Request` button.
4. Wait up to 60 seconds to receive your tPLS.

## Advanced Users: Running a PulseChain Node

If you have an archive node running on Ethereum Mainnet or PulseChain Testnet-V2B, you can save some sync time by rolling back and re-using your existing blockchain DB. See [Using An Existing Blockchain DB](#experts-only-using-an-existing-blockchain-db) below.

> **Warning**: The PulseChain Testnet includes **all** of the Ethereum mainnet state up to block `16,492,699`. This means that the system requirements for running a node will be high, particularly the storage requirements. You should only run your own testnet node if needed for development purposes, etc...

HARDWARE
- A fast CPU with 4+ cores
- 16 GB+ of RAM
- 1 TB+ free storage (SSD) for a Full Node
- 15 TB+ free storage (SSD) for an Archive Node

SOFTWARE
- [Docker](https://docs.docker.com/get-docker/) is recommended, and the commands below will tailored to running a dockerized node. By building and running the node in docker, we eliminate any environmental differences like the local golang version or the host OS.
- If you prefer, you can compile and run the executable directly, but you will need to tweak the commands below.


> **NOTE**: All commands below assume that you want to store all chain data in a local `/blockchain` directory (must have at least **1TB** free space).
>
> If needed, you can modify the commands to mount a different directory in the docker container. To do so, you will change the **absolute path** on the **left side** of the colon `:`, e.g., `docker run -v /path/to/my/dir/:/blockchain ...`
> 
> For more information see the Docker [run command reference](https://docs.docker.com/engine/reference/commandline/run/#mount-volume--v---read-only).

### 1. Prepare the Blockchain Directory

First, ensure that the intended blockchain datadir has at least 1TB of free space. The directory should be empty.

### 2. Generate JWT Secret

The HTTP connection between your beacon node and execution node needs to be authenticated using a [JWT token](https://jwt.io/). We need to save this file in the blockchain directory, to be read by both clients. There are several ways to generate this JWT token:

- Use a utility like OpenSSL to create the token via command: `openssl rand -hex 32 | tr -d "\n" > "/blockchain/jwt.hex"`.
- Use an online generator like [this](https://seanwasere.com/generate-random-hex/). Copy and paste this value into a `/blockchain/jwt.hex` file.

### 3. Start the PulseChain Execution Client

Once your blockchain directory is ready, you can start the execution client and connect to the network by providing the `pulsechain-testnet` flag. You can run either Go-Pulse (recommended) **OR** Erigon-Pulse:

Option 1: [Go-Pulse](https://gitlab.com/pulsechaincom/go-pulse)
```shell
docker run --network=host -v /blockchain:/blockchain registry.gitlab.com/pulsechaincom/go-pulse \
--pulsechain-testnet-v3 \
--authrpc.jwtsecret=/blockchain/jwt.hex \
--datadir=/blockchain/execution
```

Option 2: [Erigon-Pulse](https://gitlab.com/pulsechaincom/erigon-pulse)
```shell
docker run --network=host -v /blockchain:/blockchain registry.gitlab.com/pulsechaincom/erigon-pulse \
--chain=pulsechain-testnet-v3 \
--authrpc.jwtsecret=/blockchain/jwt.hex \
--datadir=/blockchain/execution \
--externalcl
```

> Do not Erigon's internal consensus layer at this time, passing `--externalcl` (shown above) disables this experimental feature.

### 4. Start the PulseChain Consensus Client

Once your execution client is running, you can start the consensus client and connect to the network by providing the `pulsechain-testnet` flag. You can run either Prysm-Pulse (recommended) **OR** Lighthouse-Pulse:

Option 1: [Prysm-Pulse](https://gitlab.com/pulsechaincom/prysm-pulse)
```shell
docker run --network=host -v /blockchain:/blockchain registry.gitlab.com/pulsechaincom/prysm-pulse/beacon-chain:latest \
--pulsechain-testnet-v3 \
--jwt-secret=/blockchain/jwt.hex \
--datadir=/blockchain/consensus \
--checkpoint-sync-url=https://checkpoint.v3.testnet.pulsechain.com \
--genesis-beacon-api-url=https://checkpoint.v3.testnet.pulsechain.com
```

Option 2: [Lighthouse-Pulse](https://gitlab.com/pulsechaincom/lighthouse-pulse)
```shell
docker run --network=host -v /blockchain:/blockchain registry.gitlab.com/pulsechaincom/lighthouse-pulse:latest \
--network=pulsechain_testnet_v3 \
--execution-jwt=/blockchain/jwt.hex \
--datadir=/blockchain/consensus \
--execution-endpoint=http://localhost:8551 \
--checkpoint-sync-url https://checkpoint.v3.testnet.pulsechain.com \
--http 
```

## Experts Only: Using An Existing Blockchain DB

> **Warning**: This is only valid for **archive nodes** that were sync'd with a previous version of the **PulseChain Testnet** or the **Ethereum Mainnet**.

> We will only reuse an existing execution-layer database since PulseChain starts with a new Beacon chain.

### 1. Stop the Existing Blockchain

Stop any existing Go-Pulse or Go-Ethereum processes and let the blockchain gracefully shut down.

### 2. Dump New Genesis File

Dump the `genesis.json` file from the latest Go-Pulse release. It is recommended you dump this file into your existing `--datadir` used by the previously running node. Assuming the `/blockchain` directory was being used, we can dump the updated `genesis.json` file with the command below.

```shell
docker run registry.gitlab.com/pulsechaincom/go-pulse --pulsechain-testnet dumpgenesis > /blockchain/genesis.json
```

Confirm that the `genesis.json` file has been written to your blockchain datadir. Double check the modified date to ensure this file was just created/updated.

### 3. Perform Rollback

We will need to rollback to **the last ethereum mainnet block in your DB less than `16,492,700`.
- If running an Ethereum Mainnet node, you can rollback to `16492699` (`0xFBA89B`)
- If running a PulseChain Testnet-V2B node, you will need to rollback further to `14360999` (`0xDB21A7`)

In order to rollback the chain, we need to launch the geth console to issue the `debug.setHead()` command. It's important to run with the `--nodiscover` flag to prevent the node from syncing any new blocks during this process.

```shell
docker run -it registry.gitlab.com/pulsechaincom/go-pulse --pulsechain-testnet --nodiscover console
```

**From the interactive console, verify chain state and perform the rollback:**

1. Perform the chain rollback to block the block number shown above:
    ```shell
    # for eth mainnet archive
    > debug.setHead("0xFBA89B")

    # for pulsechain-testnet-v2b archive
    > debug.setHead("0xDB21A7")
    ```

3. Verify the block number has been updated
    ```shell
    > eth.blockNumber
    16492699
    ```

3. Exit the geth console
    ```shell
    > exit
    ```

### 4. Re-Initialize Genesis w/ Updated Chain Config

With the blockchain rolled back to the last ethereum mainnet block, you can now reinitialize the genesis for Testnet V3, using the `genesis.json` file you dumped in step 2.

```shell
docker run -v /blockchain:/blockchain registry.gitlab.com/pulsechaincom/go-pulse --datadir=/blockchain init /blockchain/genesis.json
```

After the init command has completed, you can follow the normal steps above to [Advanced Users: Running a PulseChain Node](#advanced-users-running-a-pulsechain-node).
- You will use the existing execution directory for your execution node only.
- It's recommended to move the execution db data into a subdirectory such as `/blockchain/execution`.

# Staking and Running a Validator

Running a Validator is a **responsibility** to keep your node healthy and running, or your deposited funds will be at risk. A `32 million tPLS` deposit will be required to register a validator.

If you are interested in running a Validator for the PulseChain Testnet-V3 head over to our [Staking Launchpad](https://launchpad.v3.testnet.pulsechain.com).

> At this time, there is no ability to deregister and un-stake. We are following upstream progress and will be bringing the feature to PulseChain soon.
