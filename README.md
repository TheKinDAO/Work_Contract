# Work_Contract

Each Work_Contract is implemented as 2 very distinct parts:

1. As *legal code*, usually as Contract Law.
2. As *source code* usually as a Smart Contracts.

The legal code is usually stored as a digitally-signed .pdf of a legally-binding 'paper' contract.

The URI of the .pdf is required when the DAO calls `Work_Contract::nft_mint()`

The source code at `nft/src/lib.rs` is based on https://github.com/near-examples/NFT

Building this contract
======================
This will generate a WASM binaries in the `res/` directory to be later deployed onto the NEAR blockchain.
```bash
./setup.sh  # install nvm, curl, build-essential, rust, wasm32-unknown-unknown
./build.sh  # original example 
```

Testing this contract
=====================
Simple tests to verify contract is working.
```bash
cargo test -- --nocapture
```
The more complex simulation tests aren't run with this command, but we can find them in `tests/sim`.

Using this contract
===================

### Quickest deploy

You can build and deploy this smart contract to a development account. [Dev Accounts](https://docs.near.org/docs/concepts/account#dev-accounts) are auto-generated accounts to assist in developing and testing smart contracts. Please see the [Standard deploy](#standard-deploy) section for creating a more personalized account to deploy to.

```bash
near dev-deploy --wasmFile res/non_fungible_token.wasm
```

Behind the scenes, this is creating an account and deploying a contract to it. On the console, notice a message like:

>Done deploying to dev-1234567890123

In this instance, the account is `dev-1234567890123`. A file has been created containing a key pair to
the account, located at `neardev/dev-account`. To make the next few steps easier, we're going to set an
environment variable containing this development account id and use that when copy/pasting commands.
Run this command to set the environment variable:

```bash
source neardev/dev-account.env
```

You can tell if the environment variable is set correctly if your command line prints the account name after this command:
```bash
echo $CONTRACT_NAME
```

The next command will initialize the contract using the `new` method:

```bash
near call $CONTRACT_NAME new_default_meta '{"owner_id": "'$CONTRACT_NAME'"}' --accountId $CONTRACT_NAME
```

To view the NFT metadata:

```bash
near view $CONTRACT_NAME nft_metadata
```

### Standard deploy

This smart contract will get deployed to your NEAR account. For this example, please create a new NEAR account. Because NEAR allows the ability to upgrade contracts on the same account, initialization functions must be cleared. If you'd like to run this example on a NEAR account that has had prior contracts deployed, please use the `near-cli` command `near delete`, and then recreate it in Wallet. To create (or recreate) an account, please follow the directions in [Test Wallet](https://wallet.testnet.near.org) or ([NEAR Wallet](https://wallet.near.org/) if we're using `mainnet`).

In the project root, log in to your newly created account with `near-cli` by following the instructions after this command.

    near login

To make this tutorial easier to copy/paste, we're going to set an environment variable for our account id. In the below command, replace `MY_ACCOUNT_NAME` with the account name we just logged in with, including the `.testnet` (or `.near` for `mainnet`):

    ID=MY_ACCOUNT_NAME

We can tell if the environment variable is set correctly if our command line prints the account name after this command:

    echo $ID

Now we can deploy the compiled contract in this example to your account:

    near deploy --wasmFile res/non_fungible_token.wasm --accountId $ID

NFT contract should be initialized before usage. More info about the metadata at [nomicon.io](https://nomicon.io/Standards/NonFungibleToken/Metadata.html). But for now, we'll initialize with the default metadata.

    near call $ID new_default_meta '{"owner_id": "'$ID'"}' --accountId $ID

We'll be able to view our metadata right after:

    near view $ID nft_metadata

Then, let's mint our first token. This will create a NFT based on Olympus Mons where only one copy exists:

    near call $ID nft_mint '{"token_id": "0", "receiver_id": "'$ID'", "token_metadata": { "title": "Olympus Mons", "description": "Tallest mountain in charted solar system", "media": "https://upload.wikimedia.org/wikipedia/commons/thumb/0/00/Olympus_Mons_alt.jpg/1024px-Olympus_Mons_alt.jpg", "copies": 1}}' --accountId $ID --deposit 0.1

Transferring our NFT
====================

Let's set up an account to transfer our freshly minted token to. This account will be a sub-account of the NEAR account we logged in with originally via `near login`.

    near create-account alice.$ID --masterAccount $ID --initialBalance 10

Checking Alice's account for tokens:

    near view $ID nft_tokens_for_owner '{"account_id": "'alice.$ID'"}'

Then we'll transfer over the NFT into Alice's account. Exactly 1 yoctoNEAR of deposit should be attached:

    near call $ID nft_transfer '{"token_id": "0", "receiver_id": "alice.'$ID'", "memo": "transfer ownership"}' --accountId $ID --depositYocto 1

Checking Alice's account again shows us that she has the Olympus Mons token.

Notes
=====

* The maximum balance value is limited by U128 (2**128 - 1).
* JSON calls should pass U128 as a base-10 string. E.g. "100".
* This does not include escrow functionality, as ft_transfer_call provides a superior approach. An escrow system can, of course, be added as a separate contract or additional functionality within this contract.
