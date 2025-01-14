# Event Wallet

## Set Up Wallets

1. Create `packages/hardhat/ticket_secrets.csv` with all the ticket secrets. Check `packages/hardhat/ticket_secrets.csv.exampe`.
2. Run `yarn generate-from-tickets`. It'll generate `packages/hardhat/accounts.json` with a list of accounts and their private keys.

To fund the addresses make sure you have enough funds & tokens (`yarn account`)

3. Run `yarn airdrop-gas <amount> --network <network>`
4. Run `yarn airdrop-gems <amount> --network <network>`

If you want, you can generate PK QRs with the generated accounts with `yarn generate-pdfs`

## Generate NFTs

1. Add custom images to `nfts/images/`.
2. From `packages/hardhat` run `yarn generate-token-types`. This will generate random tokenTypes and save the images at `packages/nextjs/public/assets/nfts/`. This will generate the `packages/nfts/metadata.json` file too.
3. Update `packages/nextjs/assets.ts` with the new tokenTypes and the metadata (you can use the data from `packages/nfts/metadata.json` as a starting point).
4. For ipfs upload and pin data, we use Pinata. Configure your Pinata JWT key at `packages/hardhat/.env` file (*PINATA_JWT_KEY*).
5. From `packages/hardhat` run `yarn pin-metadata`. This will upload the images to ipfs, pin the files, generate the metadata for each one based on the data from `assets.ts`, and then upload and pin the metadata to ipfs. This will update the `packages/nextjs/metadataHashes.json` file (used to load the metadata and images locally instead of from ipfs), create the file `packages/nextjs/imagesPinned.json` (each ipfs hash for each tokenType), `packages/nextjs/metadatas.json` (all the metadata objects uploaded to ipfs) and `packages/nextjs/tokenMapping.txt` (the mapping from tokenType to metadata ipfs hash).
6. Copy the data from `packages/nextjs/tokenMapping.txt` to the `EventSBT` constructor, replacing the current tokenTypes.

## Contents

- [Requirements](#requirements)
- [Quickstart](#quickstart)
- [Deploying your Smart Contracts to a Live Network](#deploying-your-smart-contracts-to-a-live-network)
- [Deploying your NextJS App](#deploying-your-nextjs-app)
- [Disabling Type & Linting Error Checks](#disabling-type-and-linting-error-checks)
  - [Disabling commit checks](#disabling-commit-checks)
  - [Deploying to Vercel without any checks](#deploying-to-vercel-without-any-checks)
  - [Disabling Github Workflow](#disabling-github-workflow)
- [Contributing to Scaffold-Eth 2](#contributing-to-scaffold-eth-2)

## Requirements

Before you begin, you need to install the following tools:

- [Node (v18 LTS)](https://nodejs.org/en/download/)
- Yarn ([v1](https://classic.yarnpkg.com/en/docs/install/) or [v2+](https://yarnpkg.com/getting-started/install))
- [Git](https://git-scm.com/downloads)

## Quickstart

To get started with Scaffold-ETH 2, follow the steps below:

1. Clone this repo & install dependencies

```
git clone https://github.com/scaffold-eth/scaffold-eth-2.git
cd scaffold-eth-2
yarn install
```

2. Run a local network in the first terminal:

```
yarn chain
```

This command starts a local Ethereum network using Hardhat. The network runs on your local machine and can be used for testing and development. You can customize the network configuration in `hardhat.config.ts`.

3. On a second terminal, deploy the test contract:

```
yarn deploy
```

This command deploys a test smart contract to the local network. The contract is located in `packages/hardhat/contracts` and can be modified to suit your needs. The `yarn deploy` command uses the deploy script located in `packages/hardhat/deploy` to deploy the contract to the network. You can also customize the deploy script.

4. On a third terminal, start your NextJS app:

```
yarn start
```

Visit your app on: `http://localhost:3000`. You can interact with your smart contract using the contract component or the example ui in the frontend. You can tweak the app config in `packages/nextjs/scaffold.config.ts`.

Run smart contract test with `yarn hardhat:test`

- Edit your smart contract `YourContract.sol` in `packages/hardhat/contracts`
- Edit your frontend in `packages/nextjs/pages`
- Edit your deployment scripts in `packages/hardhat/deploy`

## Deploying your Smart Contracts to a Live Network

Once you are ready to deploy your smart contracts, there are a few things you need to adjust.

1. Select the network

By default, `yarn deploy` will deploy the contract to the local network. You can change the defaultNetwork in `packages/hardhat/hardhat.config.ts.` You could also simply run `yarn deploy --network target_network` to deploy to another network.

Check the `hardhat.config.ts` for the networks that are pre-configured. You can also add other network settings to the `hardhat.config.ts file`. Here are the [Alchemy docs](https://docs.alchemy.com/docs/how-to-add-alchemy-rpc-endpoints-to-metamask) for information on specific networks.

Example: To deploy the contract to the Sepolia network, run the command below:

```
yarn deploy --network sepolia
```

2. Generate a new account or add one to deploy the contract(s) from. Additionally you will need to add your Alchemy API key. Rename `.env.example` to `.env` and fill the required keys.

```
ALCHEMY_API_KEY="",
DEPLOYER_PRIVATE_KEY=""
```

The deployer account is the account that will deploy your contracts. Additionally, the deployer account will be used to execute any function calls that are part of your deployment script.

You can generate a random account / private key with `yarn generate` or add the private key of your crypto wallet. `yarn generate` will create a random account and add the DEPLOYER_PRIVATE_KEY to the .env file. You can check the generated account with `yarn account`.

3. Deploy your smart contract(s)

Run the command below to deploy the smart contract to the target network. Make sure to have some funds in your deployer account to pay for the transaction.

```
yarn deploy --network network_name
```

4. Verify your smart contract

You can verify your smart contract on Etherscan by running:

```
yarn verify --network network_name
```

## Deploying your NextJS App

Run `yarn vercel` and follow the steps to deploy to Vercel. Once you log in (email, github, etc), the default options should work. It'll give you a public URL.

If you want to redeploy to the same production URL you can run `yarn vercel --prod`. If you omit the `--prod` flag it will deploy it to a preview/test URL.

**Make sure your `packages/nextjs/scaffold.config.ts` file has the values you need.**

**Hint**: We recommend connecting the project GitHub repo to Vercel so you the gets automatically deployed when pushing to `main`

## Hook Example

- [useScaffoldContractRead](#usescaffoldcontractread)
- [useScaffoldContractWrite](#usescaffoldcontractwrite)
- [useScaffoldEventSubscriber](#usescaffoldeventsubscriber)
- [useScaffoldEventHistory](#usescaffoldeventhistory)
- [useDeployedContractInfo](#usedeployedcontractinfo)
- [useScaffoldContract](#usescaffoldcontract)

### useScaffoldContractRead:

Use this hook to read a value from your deployed contracts.

```ts
const { data: totalCounter } = useScaffoldContractRead({
  contractName: "YourContract",
  functionName: "getGreeting",
  args: ["ARGUMENTS IF THE FUNCTION ACCEPTS ANY"],
});
```

### useScaffoldContractWrite:

Use this hook to write to your deployed contracts.

```ts
const { writeAsync, isLoading } = useScaffoldContractRead({
  contractName: "YourContract",
  functionName: "setGreeting",
  args: ["The value to set"],
  //value if the function is payable and sends eth to it
  value: "0.01",
});
```

### useScaffoldEventSubscriber:

Use this to listen for an event emitted in the deployed smart contracts.

```ts
useScaffoldEventSubscriber({
  contractName: "YourContract",
  eventName: "GreetingChange",
  //parameters that the event emits
  //event GreetingChange(address greetingSetter, string newGreeting, bool premium, uint256 value);
  listener: (greetingSetter, newGreeting, premium, value) => {
    console.log(greetingSetter, newGreeting, premium, value);
  },
});
```

### useScaffoldEventHistory:

Use this hook to read events from a deployed contract

```ts
const {
  data: events,
  isLoading: isLoadingEvents,
  error: errorReadingEvents,
  } = useScaffoldEventHistory({
  contractName: "YourContract",
  eventName: "GreetingChange",
  fromBlock: //the block number to start reading events from,
  blockData: true,
  filters: //filters to be applied to the event (parameterName: value),
  transactionData: //if set to true it will return the transaction data for each event (default: false),
  receiptData: //if set to true it will return the receipt data for each event (default: false),
});
```

### useDeployedContractInfo:

Use this hook to get the matching contract info from the contracts file generated by yarn deploy

```ts
//contractName: name of the deployed contract
const { data: deployedContractData } = useDeployedContractInfo(contractName);
```

### useScaffoldContract:

Use to gets a deployed contract by contract name and returns a contract instance
Can also be use to read and write to the deployed smart contract

```ts
const { data: yourContract } = useScaffoldContract({
  contractName: "YourContract",
});
// will return the greeting and can be call in any function unlike useScaffoldContractRead
await yourContract?.greeting();

//can be use to write to a contract and can be called in any function
import { Signer } from "ethers";
import { useSigner } from "wagmi";

const { data: signer, isError, isLoading } = useSigner();
const { data: yourContract } = useScaffoldContract({
  contractName: "YourContract",
  signerOrProvider: signer as Signer,
});
const setGreeting = async () => {
  //call the method in any function
  await yourContract?.setGreeting("the greeting here");
};
```

## Disabling type and linting error checks

> **Hint**
> Typescript helps you catch errors at compile time, which can save time and improve code quality, but can be challenging for those who are new to the language or who are used to the more dynamic nature of JavaScript. Below are the steps to disable type & lint check at different levels

### Disabling commit checks

We run `pre-commit` [git hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) which lints the staged files and don't let you commit if there is an linting error.

To disable this, go to `.husky/pre-commit` file and comment out `yarn lint-staged --verbose`

```diff
- yarn lint-staged --verbose
+ # yarn lint-staged --verbose
```

### Deploying to Vercel without any checks

Vercel by default runs types and lint checks while developing `build` and deployment fails if there is a types or lint error.

To ignore types and lint error checks while deploying, use :

```shell
yarn vercel:yolo
```

### Disabling Github Workflow

We have github workflow setup checkout `.github/workflows/lint.yaml` which runs types and lint error checks every time code is **pushed** to `main` branch or **pull request** is made to `main` branch

To disable it, **delete `.github` directory**

## Contributing to Scaffold-Eth 2

We welcome contributions to Scaffold-Eth 2!

Please see [CONTRIBUTING.MD](https://github.com/scaffold-eth/se-2/blob/master/CONTRIBUTING.md) for more information and guidelines for contributing to Scaffold-Eth 2.
