---
layout: rsk
title: Getting Started
tags: rif, enveloping, rsk, gas station network, gsn, defi, getting-started
---

Prepare the environment by following the instructions in [Setup](/rif/enveloping/dev-setup/).

## Building the project

Clone the project.

```shell
git clone https://github.com/rsksmart/enveloping
```

Then run the following from the Enveloping project's root directory

```shell
yarn install
yarn prepare
```

> See the [Yarn Installation Guide](https://yarnpkg.com/getting-started/install) to install `yarn globally`.

### Docker Build

To build the default `PAPYRUS-2.1.0`, run the command below in the `rsknode` folder.

```shell
docker build -t rskj:2.1.0-PAPYRUS .
```

### Run Docker

To run, inside the `rsknode` folder, use the recently generated tag (`-t` parameter for `docker build`):

```shell
docker run -p 127.0.0.1:4444:4444 --name enveloping-rskj rskj:2.1.0-PAPYRUS --regtest
```

`--regtest` can be modified to any other of the networks or parameters supported by the RSKj binary.

> See more instructions on [How To build a different version](https://github.com/rsksmart/enveloping/blob/master/rsknode/README.md) (optional: it runs an RSK node)

## Deploy contracts locally

We use `truffle` for deploying contracts.

Open another terminal in the enveloping root directory, and run the command below;

```shell
npx truffle migrate --network rsk
```

> NOTE: We are using the RSK [Regtest network](https://developers.rsk.co/rsk/node/configure/switch-network/#regtest), and not one of the public networks.

## Run a relay server locally

In order to run an instance of Enveloping in Regtest:

(1) From the `jsrelay` directory

```shell
npx webpack
```

(2) From the root directory, run:

```shell
node dist/src/cli/commands/enveloping.js boot-test --network http://localhost:4444/
```
Specifying `localhost:4444` connects to an RSK Regtest node.

To check if the `jsrelay` server is working, run:

```shell
curl http://localhost:<PORT>/getaddr
```

`<PORT>`: Port is where the `jsrelay` is running

Output:

```json
{
  "relayWorkerAddress":"0x75b701edf0a4abcf15e68e8aefe85bf416c0c678", "relayManagerAddress":"0x05029db78d8aa9e8a7fa8813f72dffe37d15aa64","relayHubAddress":"0x463F29B11503e198f6EbeC9903b4e5AaEddf6D29",
  "minGasPrice":"1",
  "chainId":"33",
  "networkId":"33",
  "ready":true,
  "version":"2.0.1"
}
```

(3) Deploy contracts on Testnet

We use `truffle` for deploying contracts. 

In the enveloping root directory, run the command below;

```shell
npx truffle migrate --network rsktestnet
```

> Note: To use Testnet, you should have an unlocked account with funds or configure it in `truffle.js`.

We have already deployed these contracts on Testnet. See [Launching enveloping](https://github.com/rsksmart/enveloping/blob/master/docs/launching_enveloping.md#testnet-contracts).

(4) Run a Relay Server on Testnet

In order to run an Enveloping instance in Testnet,
clone the project then run the following
from the project's root directory:

1.  Create the project home folder,
    in this folder, the `jsrelay` databases will be placed:
    ```shell
    mkdir enveloping_relay
    ```
2.  In a terminal, run
    ```shell
    node dist/src/cli/commands/enveloping.js \
      relayer-run \
      --rskNodeUrl "http://localhost:4444" \
      --relayHubAddress=<RELAY_HUB_CONTRACT_ADDRESS> \
      --deployVerifierAddress=<DEPLOY_VERIFIER_CONTRACT_ADDRESS> \
      --relayVerifierAddress=<RELAY_VERIFIER_CONTRACT_ADDRESS> \
      --versionRegistryAddress=<VERSION_REGISTRY_CONTRACT_ADDRESS> \
      --url <RELAY_URL> \
      --port 8090 \
      --workdir enveloping_relay \
      --checkInterval 90000
    ```
    where;
    - `<RELAY_HUB_CONTRACT_ADDRESS>`:
      is the address for the Relay Hub you are using in the current network
    - `<DEPLOY_VERIFIER_CONTRACT_ADDRESS>`:
  is the address for the Deploy Verifier you are using in the current network
    - `<RELAY_VERIFIER_CONTRACT_ADDRESS>`:
    is the address for the Relay Verifier you are using in the current network
    - `<VERSION_REGISTRY_CONTRACT_ADDRESS>`:
    is the address for the Version Registry you are using in the current network
    
      ([see Testnet Contracts section](https://github.com/rsksmart/enveloping/blob/master/docs/launching_enveloping.md#c02.1)),
    - `<RELAY_URL>`:
      in most cases will be `http://localhost`,
      and the server will be reachable at
      `<RELAY_URL>:port` unless
      `<RELAY_URL>` already defines a port
      (e.g, if `<RELAY_URL>` is `http://localhost:8090/jsrelay`)
3.  In another terminal, run
    ```shell
    curl http://localhost:8090/getaddr
    ```
    which will return some JSON with information
    of the running `jsrelay` server, for example:
    ```json
    {"relayWorkerAddress":"0xe722143177fe9c7c58057dc3d98d87f6c414dc95","relayManagerAddress":"0xe0820002dfaa69cbf8add6a738171e8eb0a5ee54", "relayHubAddress":"0x38bebd507aBC3D76B10d61f5C95668e1240D087F", "minGasPrice":"6000000000", "chainId":"31", "networkId":"31","ready":false,"version":"2.0.1"}
    ```
4. Send at least 0.001 tRBTC to `relayManagerAddress` to set it up
5. Send at least 0.001 tRBTC to `relayWorkerAddress` to set it up
6.  Once both addresses have been funded, run;
    ```shell
    node dist/src/cli/commands/enveloping.js \
      relayer-register \
      --network <RSKJ_NODE_URL> \
      --hub <RELAY_HUB_CONTRACT_ADDRESS> \
      -m secret_mnemonic \
      --from <ADDRESS> \
      --funds <FUNDS> \
      --stake <STAKE> \
      --relayUrl <RELAY_URL>
    ```
    where `secret_mnemonic` contains the path to
    a file with the mnemonic of the account
    to use during the relay server registration,
    and `<ADDRESS>` is the account address associated to that mnemonic.
    Wait until the relay server prints a message saying `RELAY: READY`.

## Troubleshooting

> Running some test and one of them throws:
>
> ```
> Error: listen EADDRINUSE: address already in use :::8090
> ```

This means that the relay server is running in the background.
Run the bash file `scripts/kill-relay-server.sh`
to kill it and make that port available again.
