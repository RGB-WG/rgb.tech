+++
title = "RGB Regtest Environment"
weight = 10
[extra]
anchor = "cmd"
bg-color = "green"
+++

A ready-to-use Bitcoin regtest environment required for testing the open-source RGB project by BitLight, featuring:

- Deep integration of the Esplora UI browser built into version v0.11
- A descriptor address generation tool required for the RGB CLI
- A Bitcoin wallet faucet tool

### Prerequisites

Before installing the RGB regtest environment, you need to install the components used in its build process.

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- Make

## Getting Started

Clone repo [bitlightlabs/bitlight-local-env-public](https://github.com/bitlightlabs/bitlight-local-env-public)

```shell
git clone git@github.com:bitlightlabs/bitlight-local-env-public.git
cd bitlight-local-env-public
```

check the `.env` file and change the values if needed

Run `make start` or `make up` to start the development environment

## Explore the environment

### Esplora

Esplorea API is running on `http://localhost:3002`, you can check it by running `curl localhost:3002/mempool | jq` for
example:

```shell
$ curl -s localhost:3002/mempool | jq
{
  "count": 0,
  "vsize": 0,
  "total_fee": 0,
  "fee_histogram": []
}
```

The Esplora UI is running on `http://localhost:5002`, you can open it in your browser to see the Esplora UI

### Bitcoin Wallet Faucet

You can mint and send bitcoins to your wallet using the Bitcoin Wallet Faucet. To use it, run the following commands:

```shell
$ make core-cli
# (in the docker container)
/cli mint 1 # mint 1 bitcoin
/cli send <address> <amount> # send <amount> bitcoin to the <address>
```

Example:

```shell
$ make core-cli
docker-compose -p bitlight-local-env exec -it -w /cli bitcoin-core /cli/active.sh
/cli $ load_wallet
load wallet's address: bcrt1qv6428v9lzk9ac5aswp4hlaf773r48824stfvx9 with balance: 50.00000000
/cli $ send bcrt1pr3rupmav8a7av7dqfyvynu2wk02lduggnh9ln4ndze9aqvuv9y3sklwrss 25
load wallet's address: bcrt1qx4mwpxpw0d8prsrw4fg8q2s94au6s2qk5dwc9g with balance: 50.00000000
9b2eca8ba85f2e11e97c820bfc5990a20387812f313cea6e8889c309ed8bff45
Sent 25 to bcrt1pr3rupmav8a7av7dqfyvynu2wk02lduggnh9ln4ndze9aqvuv9y3sklwrss
```

### Wallet Service Setup and Usage

There are two wallets available in the environment: Alice and Bob. You can add more by yourself. To use the wallets, run the following commands:

```shell
$ make alice-cli # or make bob-cli
# (in the docker container with Alice wallet REPL console)
> help # to see the available commands
> wallet sync # to sync the wallet with the blockchain
> wallet balance # to check the wallet balance
> wallet list_unspent # to list the unspent transactions
> wallet get_new_address # to get a new address
```

For more usage and example, please refer to the [Wallets Usage](https://github.com/bitlightlabs/bitlight-local-env-public?tab=readme-ov-file#wallets)

### E2E Tests

You can use it in you E2E tests. For example, there a test in the `tests/e2e.rs` and run with GitHub actions:

```yaml
# .github/workflows/test.yml
      - name: Build Docker images
        run: |
          docker-compose -f docker-compose.yml build
      - name: Start Blockchain
        run: |
          make up
        env:
          API_PORT: 3002
          RPC_USER: bitcoin
          RPC_PASSWORD: bitcoin

      - name: Build
        run: make build  # build the project by your own

      - name: Run tests
        run: make test # run the tests
```
