## 🤔Educational testing task

### Task:
- Create a digital token named STUDY on a Cosmos test network `SEI Testnet` or `Neutron Testnet`.  
- Initialize and Mint 100,000 CW20 Token (STUDY coins) using CW20 Token Standard repository.
- Use the provided GoLang Smart Contract repository to create the token contract.
- Build and Deploy CW20 Merkle Airdrop contract using CW20 Merkle Airdrop repository.
- Airdrop 100,000 STUDY coins to `neutron1wjlwn3sd09p2gh00pvumvhynlkmrl0zj6ssj6g` testnet address.  

### Resources:
- [CosmWasm GitHub Repository](https://github.com/CosmWasm/cw-plus)
- [CW20 Token Standard Repository](https://github.com/CosmWasm/cw-plus/tree/main/contracts/cw20-base)
- [CW20 Merkle Airdrop Repository](https://github.com/CosmWasm/cw-plus/tree/0.9.x/contracts/cw20-merkle-airdrop)
- [Neutron faucet](https://docs.neutron.org/neutron/faq/#where-is-the-testnet-faucet)
- [Sei faucet](https://www.docs.sei.io/getting-tokens/faucets)

***

### Preparation:
- Install Go using `go.dev/doc/install` guide.
- Install Rust using `rust-lang.org/tools/install` guide.
- [Optional] Install vscode and add `github.com/oraichain/cw-ide-vscode` extension to have predefined commands `build`, `deploy` etc.
- [Optional] Add Go and Rust extensions to vscode.
- Install `wasmd` using `github.com/CosmWasm/wasmd` guide, change default wallet prefix to `neutron`.
- [Optional] In case of installation errors on Ubuntu `sudo apt update` and `sudo apt install build-essential`
- Run `cargo install cargo-generate` to install cargo-generate.
- Run `rustup target add wasm32-unknown-unknown` to add wasm32-unknown-unknown target.
- install Keplr wallet as Chrome extension.
- Add with `https://dev-wallet.keplr.app/chains/neutron` neutron testnet chain to Keplr wallet.
- Use telegram faucet to get some tokens with command `/request <NEUTRON_ADDRESS>`.


- Install yarn package manager using `yarnpkg.com/getting-started/install` guide.

### Links
- [Neutron Testnet Block Explorer](https://neutron.celat.one/pion-1)
- [Telegram Neutron Faucet](https://t.me/+SyhWrlnwfCw2NGM6)
- [Neutron RPC Node](https://rpc-falcron.pion-1.ntrn.tech)

## Mint CW20 Token on Neutron testnet with default cw20-base contract and wasmd:
- Clone CosmWasm CW20 Contracts repository.  
    `https://github.com/CosmWasm/cw-plus.git`
- Navigate to `contracts/cw20-base` directory, it contains predefined contract files.
- Run to build wasm contract file.
  ```
  RUSTFLAGS='-C link-arg=-s' cargo wasm
  cp ../../target/wasm32-unknown-unknown/release/cw20_base.wasm .
  ls -l cw20_base.wasm
  sha256sum cw20_base.wasm
  ```

- Or run in root folder of cloned repository 
  ```
  docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/optimizer:0.16.0
  ```
- Save `cw20_base.wasm` file in suitable place for further usage.  

- Run `wasmd init mynode --chain-id=pion-1` to initialize wasmd config.
- Recover wallet with wasmd from seed phrase.
  `wasmd keys add <WALLET_NAME> --recover`
- Upload your cw20_base.wasm smart contract. 
  ```
  wasmd tx wasm store ./cw20_base.wasm --from <WALLET_NAME> --chain-id pion-1 \
  --gas auto --gas-prices 0.025untrn --gas-adjustment 1.25 -y --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Save received tx hash.
- Run to check transaction details by tx hash.   
  `wasmd query tx <TX_HASH> --node https://rpc-falcron.pion-1.ntrn.tech`
- Find row `key: code_id"` and save this value. It's contract code ID.
- Instantiate your uploaded contract with payload. 
  ```
  wasmd tx wasm instantiate <CODE_ID> \
  '{"name":"<TOKEN_NAME>","symbol":"<TOKEN_SYMBOL>","decimals":<DECIMALS>,"initial_balances":[{"address":"<YOUR_ADDRESS>","amount":"10000000"}], \
  "mint":{"cap":"<CAP>","minter":"<YOUR_ADDRESS>"}}' \ 
  --from <WALLET_NAME> --label "<SOME_LABEL>" --chain-id pion-1 --gas auto --gas-prices 0.025untrn \ 
  --gas-adjustment 1.25 -y --node https://rpc-falcron.pion-1.ntrn.tech --no-admin
  ```
- To get list of contract addresses:   
  `wasmd query wasm list-contract-by-code <CONTRACT_ID> --node https://rpc-falcron.pion-1.ntrn.tech`
- To check contract balance use:  
  `wasmd query wasm contract-state smart <MINT_CONTRACT_ADDRESS> '{"balance":{"address":"<MINTER_WALLET>"}}' --node https://rpc-falcron.pion-1.ntrn.tech`


### Result:
- Block explorer:   
  ```
  https://neutron.celat.one/pion-1/accounts/neutron1c2uj8a3pmvd79q5vr2j8ajl4gr7ru2huehh7mk
  ```
- Symbol `STTS`, name `ST Token`, decimals `6`, amount `10000000000000`, address `neutron19n0myu7nczlspmt6z6jzhaypm0p6ncsq0hngku4c605pzlz9ptvq9dhu79`.
- Contract code ID `5510`.
- Contract tx hash `F46C1572CC2C63597F429650FDABF5FE915DE6853482E9703586EE21FE0A712E`.
- Contract address `neutron19n0myu7nczlspmt6z6jzhaypm0p6ncsq0hngku4c605pzlz9ptvq9dhu79`.
- Run to check mint contract balance:    
  ```
  wasmd query wasm contract-state smart neutron19n0myu7nczlspmt6z6jzhaypm0p6ncsq0hngku4c605pzlz9ptvq9dhu79 \
  '{"balance":{"address":"neutron1c2uj8a3pmvd79q5vr2j8ajl4gr7ru2huehh7mk"}}' --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Run to check mint contract token details:    
  ```
  wasmd query wasm contract-state smart neutron19n0myu7nczlspmt6z6jzhaypm0p6ncsq0hngku4c605pzlz9ptvq9dhu79 \
  '{"token_info":{}}' --node https://rpc-falcron.pion-1.ntrn.tech
  ```

## Merkle Airdrop on Neutron test network with default cw20-merkle-airdrop contract and wasmd:
- Clone non-official repository with Merkle Airdrop contract. Because 0.9.x branch has obsolete code base and dependencies,    
  it would take a lot of time to fix everything.
  ```
  https://github.com/thr2240/merkle-airdrop.git
  ```
- Run in root folder of cloned repository `cargo update`
- Run in root folder of cloned repository:
  ```
  docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/optimizer:0.16.0
  ```
- Save builder wasm contract in suitable place.
- Navigate to helpers folder and run `yarn install` to install deps for Merkle tree build.
  - [Optional] On Ubuntu be sure to change line endings in `bin/run` file to LF.
  - Fill `airdrop_list.json` with airdrop addresses and amounts.
  - Run in helper folder `./bin/run generateRoot --file ./airdrop_list.json` to generate Merkle tree root.
  - Save generated root hash.
  - To get proof for specific wallet run:  
  ```
  ./bin/run generateProofs --file ../testdata/airdrop_list.json --address <WALLET_TO_CHECK> --amount <AIRDROP_AMOUNT>
  ```

- Upload your wasm airdrop contract: 
  ```
  wasmd tx wasm store ./cw20_merkle_airdrop.wasm --from <WALLET_NAME> --chain-id pion-1 --gas auto \
  --gas-prices 0.025untrn --gas-adjustment 1.25 -y --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Save received tx hash.
- Run `wasmd query tx <TX_HASH> --node https://rpc-falcron.pion-1.ntrn.tech` to get contract data.
- Find row `key: code_id"` and save this value. It's contract code ID.
- Instantiate your airdrop contract with payload:
  ```
  wasmd tx wasm instantiate <CONTRACT_ID> '{"owner":"<WALLET_ADDR>", "cw20_token_address": "<MINT_CONTRACT_ADDR>"}' \ 
  --from <WALLET_NAME> --label "practice_airdrop" --chain-id pion-1 --gas auto --gas-prices 0.025untrn \ 
  --gas-adjustment 1.25 -y --node https://rpc-falcron.pion-1.ntrn.tech --no-admin
  ```

- Check your contract address or use block explorer.  
  ```
  wasmd query wasm list-contract-by-code <CONTRACT_ID> --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Transfer tokens for further airdrop: 
  ```
  wasmd tx wasm execute <CONTRACT_WITH_MINTED TOKENS> '{"transfer":{"recipient":"<AIRDROP_CONTRACT>","amount":"<AMOUNT>"}}' \
  --from <YOUR_WALLET> --chain-id pion-1 --gas auto --gas-prices 0.025untrn --gas-adjustment 1.4 -y --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Register Merkle root in smart contract:
  ```
  wasmd tx wasm execute <CONTRACT_ADDRESS> '{"register_merkle_root": {"merkle_root": "<MERKLE_ROOT>"}}' \
  --from <WALLET_NAME> --chain-id pion-1 --gas auto --gas-prices 0.025untrn --gas-adjustment 1.25 -y \
  --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- [Optional] Update root for further airdrop stages:  
  ```
  wasmd tx wasm execute <CONTRACT_ADDRESS> \
  '{"register_merkle_root": {"merkle_root": "<MERKLE_TREE_ROOT>", "start": {"at_height": <BLOCK_HEIGHT>}, "total_amount": "<AMOUNT", "hrp": "neutron"}}' \
  --from nu --chain-id pion-1 --gas auto --gas-prices 0.025untrn --gas-adjustment 1.4 -y --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Claim tokens with wallet which takes part in airdrop: 
  ```
  wasmd tx wasm execute <CONTRACT_ADDRESS> '{"claim":{"stage":1,"amount":"<CLAIM_AMOUNT>","proof":["<CLAIN_WALLET"]}}' \
  --from n --chain-id pion-1 --gas auto --gas-prices 0.025untrn --gas-adjustment 1.3 -y --node https://rpc-falcron.pion-1.ntrn.tech
  ```

### Result:
- Block explorer: 
  ```
  https://neutron.celat.one/pion-1/accounts/neutron1c2uj8a3pmvd79q5vr2j8ajl4gr7ru2huehh7mk
  ```
- Contract tx hash `B123F9FE38898231BD9944A4ADAB1D0CB340802AD0B78896DEA88DEF584F33A4`.
- Check contract data: 
  ```
  wasmd query tx B123F9FE38898231BD9944A4ADAB1D0CB340802AD0B78896DEA88DEF584F33A4 --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Contract code ID `5545`.
- Airdrop Contract address `neutron1shpnqp7dz4tcac2h930lvzy3z20yysmfv8px6yqqcda5w08wk92sx0tdaz`
- Merkle tree root `3aa98e6ddb24ba2dadf60f0d558e01fa30e4961ecd563864af4f82d2beaf9d9e`
- Proof `3b49181ad70ba8418dadd47caec364bbfca1691acecbd94f8693ff81c7ea7bb4` for `"address": "neutron1ac88e2tu6wlszq56fvnypq0ns8quc5dj3rjtvd", "amount": "9999"`
- Check airdrop contract balance:
  ```
  wasmd query wasm contract-state smart neutron19n0myu7nczlspmt6z6jzhaypm0p6ncsq0hngku4c605pzlz9ptvq9dhu79 \
  '{"balance":{"address":"neutron1shpnqp7dz4tcac2h930lvzy3z20yysmfv8px6yqqcda5w08wk92sx0tdaz"}}' \
  --node https://rpc-falcron.pion-1.ntrn.tech
  ```
- Check airdropped wallet balance: 
  ```
  wasmd query wasm contract-state smart neutron19n0myu7nczlspmt6z6jzhaypm0p6ncsq0hngku4c605pzlz9ptvq9dhu79 \
  '{"balance":{"address":"neutron1ac88e2tu6wlszq56fvnypq0ns8quc5dj3rjtvd"}}' \
  --node https://rpc-falcron.pion-1.ntrn.tech
  ```