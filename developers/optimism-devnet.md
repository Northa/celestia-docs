---
description: Start your own devnet with a modified version of optimism-bedrock.
---

# Deploy an OP Stack devnet

This guide will show you how to run your own OP Stack devnet.

## Dependency setup

### Environment setup and Golang installation

First, [install dependencies for Celestia software](../nodes/environment.md)
and for [OP Stack](https://community.optimism.io/docs/developers/build/dev-node/).

### Clone repository

Next, clone the repo:

```bash
cd $HOME
git clone https://github.com/celestiaorg/optimism
cd optimism
```

Check out to the version for either the stable version or upstream version:

::: code-group

```bash-vue [v1.0.0-OP_v1.2.0-CN_v0.12.0]
git checkout tags/v1.0.0-OP_v1.2.0-CN_v0.12.0
git submodule update --init --recursive
```

```bash-vue [celestia-develop]
git checkout celestia-develop
git submodule update --init --recursive
```

:::

## Build devnet

Build TypeScript definitions for TS dependencies:

```bash
cd $HOME
cd optimism
make
```

Set environment variables to start network:

```bash
export SEQUENCER_BATCH_INBOX_ADDRESS=0xff00000000000000000000000000000000000000
export L2OO_ADDRESS=0x70997970C51812dc3A010C7d01b50e0d17dc79C8
```

### Start devnet

Start the network by running:

```bash
make devnet-up
```

This starts up the layer 1 (ETH), layer 2 (`op-geth`), data availability
layer (Celestia), the sequencer (`op-node`), batch submitter (`op-batcher`),
state commitment service (`op-proposer`).

### View the logs of the devnet

If you'd like to view the logs of the devnet, run the following command
from the root of the Optimism directory:

```bash
make devnet-logs
```

::: details Optional: Docker tips

### Stop devnet

If you'd like to start the network over, use the following command
to safely shut down all of the containers:

```bash
make devnet-down
```

Then clean out the old config:

```bash
make devnet-clean
```

### Viewing containers

To view the containers running, send:

```bash
docker ps
```

Find the container ID of the `ops-bedrock_op-batcher_1`
and run the following to follow the logs:

```bash
docker logs -f <container-id>
```

In a new terminal, find the container ID of the
`ghcr.io/celestiaorg/local-celestia-devnet:main`
and run the following to follow the logs:

```bash
docker logs -f <container-id>
```

You can do the same for other containers as you
explore the stack.

:::

## Find a transaction

Now, we'll check for a recent transaction on the L1 with:

```bash
cast block latest --rpc-url localhost:8545
```

Output of a block that contains a transaction will look like this:

```console
baseFeePerGas        7
difficulty           2
extraData            0xd883010d04846765746888676f312e32312e33856c696e7578000000000000006b3afa42dce1f87f1f07a1ef569c4d43e41738ef93c865098bfa1458645f384e2e4498bcfe4ad9353ff1913a2e16162f496fafe5b0939a6c78fb5b503248d6da01
gasLimit             30000000
gasUsed              21568
hash                 0x1cb54d2369752ef73511c202ff9cdfd0eadf3a77b7aef0092bea63f2b5d57659
logsBloom            0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
miner                0x0000000000000000000000000000000000000000
mixHash              0x0000000000000000000000000000000000000000000000000000000000000000
nonce                0x0000000000000000
number               1141
parentHash           0x664bf4bb4a57dd5768a0a98991d77c58fb7a4e164c2581c79fb33ce9c3d4c250
receiptsRoot         0xaf8ff6af1180c8be9e4e8f3a5f882b3b227233f4abbefa479836d3721682a389
sealFields           []
sha3Uncles           0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347
size                 767
stateRoot            0xd4b998a35d20d98ed3488221f0c161a0a9572d3de66399482553c8e3d2fae751
timestamp            1699638350
withdrawalsRoot
totalDifficulty      2283
transactions:        [
  0x79a0a7a1b4936aafe7a37dbfb07a6a9e55c145a4ed6fd54f962649b4b7db8de7
]
```

Copy the transaction hash from `transactions: <transaction-hash>` and
set it as a variable:

```bash
export TX_HASH=0x79a0a7a1b4936aafe7a37dbfb07a6a9e55c145a4ed6fd54f962649b4b7db8de7
```

## Read the transaction call data

Now read the transaction call data on the L1:

```bash
cast tx $TX_HASH --rpc-url localhost:8545
```

The output will look similar to below:

```console
blockHash            0x9f4dfae061b5ddd86f95a81be5daa0d7fe32e7f7f770f86dc375e0007d249bd2
blockNumber          24
from                 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC
gas                  21572
gasPrice             1040676758
hash                 0xadd3a5dc0b8c605aeac891098e87cbaff43bb642896ebbf74f964c0690e46df2
input                0xce3500000000000000769074a923011bdda721eacc34c8a77c69c10f2b6c8e659f987e82f217a5340f
nonce                4
r                    0xaf5c1505c7dfcebca94d9a6a8c0caf99b6c87a8ed6d6c0b3161c9026f270a84f
s                    0x383ed2debf9f9055920cd7340418dda7e2bca6b989eb6992d83d123d4e322f2a
to                   0xFf00000000000000000000000000000000000901
transactionIndex     0
v                    0
value                0
yParity              0
```

::: tip
You are looking for a batcher transaction to the address
`0xFf00000000000000000000000000000000000901`.
:::

First, remove the prefix `0xce`.
Now, set the `input` as the `INPUT` variable and encode it as
base64:

```bash
export INPUT=ce3500000000000000769074a923011bdda721eacc34c8a77c69c10f2b6c8e659f987e82f217a5340f
export ENCODED_INPUT=$(echo "$INPUT" | xxd -r -p | base64)
```

:::tip
Remember to remove the `0xce` prefix!
:::

## Find the data on Celestia

Clone the `go-da` repository:

```bash
cd $HOME
git clone https://github.com/rollkit/go-da.git
cd go-da/proto/da
```

Now, from `go-da/proto/da` run:

<!-- markdownlint-disable MD013 -->

```bash
grpcurl -proto da.proto -plaintext -d "{\"ids\": [{\"value\": \"$ENCODED_INPUT\"}]}" 127.0.0.1:26650 da.DAService.Get
```

Your result will look similar to the below!

```console
{
  "blobs": [
    {
      "value": "AKUumhJ8FnuyVrBs38FDKEIAAAAAAZB42trhw/DDc4GFAlv4klkv5Zh4E16mmO5fpNOS1f5wzpds8YK3S0Rvs4ULLJj13euw+Ovdv6Q23zuV1ShROEvk5aptIT7bGmZunvc1OiKwJTXVbN0BiGm6k2zNWq78cNsT2ez3+nzQq84Ds28or/aKz/o1w4NpV7w4caZtgJomX71w96m63+xzYnarXLu7WWvRrwbeb6cW8R93YHXt1r4+TXCBGVe76obzf5JLTNu22gksD2cL+83D8DGjX0FKcwZD0VofkGmboKY1uTddu8704s2MwgNNe09s1bzw+n9Fq6fKFw7pvwJL200eCS0oFJ3HfPAEywnlgyyGQc89dh+98GD5TrdU4aNql9afmW+sDzJtC9S0fzLWYROOS0bvK3W7EvNpmWXe5qrdzKlBmv1LZi4ofrrxLHGmbYOaJhHsEn+B81lGUh33HDet8K9nVKKSF2+W3Xul6uPSxydPBwsv2GHskR+yfUlDbvyl1ROTvtS1zXlpEPz0M1e/RIIt57fVj0Gm7TgACAAA//+Qdel2AQ=="
    }
  ]
}
```

## ETH fallback

The [ETH fallback mechanism](eth-fallback.md) allows rollups to "fall back" to
Ethereum or another EVM chain in the case of downtime or errors submitting
data to Celestia.

Testing out the ETH fallback mechanism can be done
with the `go-da` tool. Triggering a simultaneous blob transaction will
cause the `op-batcher` blob transaction to fail, with an `incorrect account
sequence` error, which triggers a fallback to Ethereum.

To trigger the transaction, send this command from the same `go/proto/da` directory:

```bash
grpcurl -proto da.proto -plaintext -d '{"blobs": [{"value": "SGVsbG8gd28ybGQh"}]}' 127.0.0.1:26650 da.DAService.Submit
```

Alternatively, you can shut off the `local-celestia-devnet` and see that
the OP Stack devnet logs show that the rollup has fallen back to the L1,
in this case Ethereum, for posting data.
