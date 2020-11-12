---
layout: rsk
title: Mainnet Guide
tags: rsk, rbtc, conversion, peg, 2-way, peg-in, peg-out, federation
description: 'Converting BTC to R-BTC (peg-in) and R-BTC to BTC (peg-out).'
collection_order: 3110
permalink: /rsk/rbtc/conversion/networks/mainnet/
---

In this section we will go over the steps of converting BTC to R-BTC and vice versa in Bitcoin and RSK Mainnets.

> Note: The minimum amount of Bitcoin to convert is **0.01 BTC** for Mainnet.

## BTC to R-BTC conversion

Instructions on how to do a Mainnet peg-in.

**1 Get a BTC address with balance**

Any Bitcoin wallet that supports legacy (`p2pkh`) private key works for this step,
and here we recommend to use Electrum BTC wallet for connecting to BTC Mainnet.

- Download the wallet from [Electrum Website](https://bitzuma.com/posts/a-beginners-guide-to-the-electrum-bitcoin-wallet/)
- Install Electrum
- Start Electrum
- Once Electrum starts, create or import a wallet
- Go to the third tab "Receive". You will see a Bitcoin Testnet address like below.

> Note: The Bitcoin wallet needs to be legacy (not Segwit)
> whose public key starts with either `m` or `n`,
> and private key starting with `p2pkh:`

![Create a Legacy (`p2pkh`) wallet](/dist/images/legacy-private-key.png)

**2 Send Bitcoin to RSK Federation address**

<div class="fade alert alert-warning show">
Note: You need to send a minimum amount of 0.01 BTC.
</div>

The RSK Federation address is retrieved by making a Smart Contract call
on RSK Mainnet. In order to make the call, you will need to have
[MyCrypto](https://mycrypto.com/contracts/interact) installed,
select RSK Network, and Navigate to
_"MyCrypto -> Contracts -> Select Existing Contracts -> "Bridge" -> "getFederationAddress"_
to execute the call.
It should look like the screenshot below.

![Get RSK Federation address from MyCrypto](/dist/images/mycrypto-federation.png)

Once you have the RSK Federation address, you can send Bitcoin to it from your Bitcoin address.

**3 Wait for BTC confirmations**

To ensure the transaction, we need to wait 100 BTC confirmations, be patient :)

> 100 blocks \* 10 minutes/block = 1000 minutes = 16.667 hours approx.

**4 Get R-BTC address with BTC private key**

You can get a corresponding R-BTC address from your BTC private key by using [github.com/rsksmart/utils](https://github.com/rsksmart/utils). If you do not want to compile the utility, you can download the [latest release](https://github.com/rsksmart/utils/releases/latest).

> Note: when entering Bitcoin private key do not include _p2pkh:_ in the front.

**5 Check R-BTC balance**

You can check balance of R-BTC address on Metamask, MyCrypto,
or any RSK compatible wallets.

> Note: You have to wait a minimum of 100 confirmations +
> a minimum of 5 minutes for checking your R-BTC balance

## R-BTC to BTC conversion

Instructions on how to do a Mainnet peg-out.

**1 Get BTC address with R-BTC private key**

You can get a corresponding BTC address from your R-BTC private key by using [github.com/rsksmart/utils](https://github.com/rsksmart/utils). If you do not want to compile the utility, you can download the [latest release](https://github.com/rsksmart/utils/releases/latest).

**2 Send R-BTC to RSK Bridge Contract**

RSK Bridge Contract address: `0x0000000000000000000000000000000001000006`

> Note: The minimum amount to send is 0.008 R-BTC for Mainnet
> Gas Limit of the transaction needs to be manually set at 100,000 gas;
> otherwise the transaction will fail. Gas Price can be set to 0.06 gwei.

![Customize Gas in Metamask before send transaction on RSK](/dist/images/metamask-gas-limit.png)

**3 Check balance of BTC address**

You can either use Electrum wallet downloaded earlier or from any
Bitcoin explorer to check the balance.

> Note: The release process on Bitcoin network takes
> 4000 RSK block confirmations and at least 10 more minutes.
