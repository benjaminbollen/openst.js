OpenST.js
============

![Build master](https://img.shields.io/travis/OpenSTFoundation/openst.js/master.svg?label=build%20master&style=flat)
![Build develop](https://img.shields.io/travis/OpenSTFoundation/openst.js/develop.svg?label=build%20develop&style=flat)
![npm version](https://img.shields.io/npm/v/@openstfoundation/openst.js.svg?style=flat)
![Discuss on Discourse](https://img.shields.io/discourse/https/discuss.openst.org/topics.svg?style=flat)
![Chat on Gitter](https://img.shields.io/gitter/room/OpenSTFoundation/SimpleToken.svg?style=flat)

OpenST.js supports interaction with openst-contracts.

##  Setup
This library assumes that nodejs and geth are installed and running. To install OpenST.js in your project using npm:

```bash
$ npm install @openstfoundation/openst.js --save
```

This code was tested with geth version: 1.7.3-stable. Other higher versions should also work.

## Creating a OpenST object
The OpenST object is an entry point: using the OpenST object, a staking can be initiated.

```js
// Creating OpenST.js object
const OpenST = require('@openstfoundation/openst.js');

```

## ABI and BIN provider

openst.js comes with an abi-bin provider for managing abi(s) and bin(s).

The abiBinProvider provides abi(s) and bin(s) for the following contracts:

* [TokenHolder](https://github.com/OpenSTFoundation/openst-contracts/blob/0.10.0-alpha.1/contracts/token/TokenHolder.sol) (TokenHolder contract deployed on UtilityChain)
* [TokenRules](https://github.com/OpenSTFoundation/openst-contracts/blob/0.10.0-alpha.1/contracts/token/TokenRules.sol) (TokenRules contract deployed on UtilityChain)
* [PricerRule](https://github.com/OpenSTFoundation/openst-contracts/blob/0.10.0-alpha.1/contracts/rules/PricerRule.sol) (PricerRule is a rule contract deployed on UtilityChain)
* [GnosisSafe](https://github.com/gnosis/safe-contracts/blob/v0.1.0/contracts/GnosisSafe.sol) (MultiSignature wallet with support for confirmations using signed messages)
* [DelayedRecoveryModule](https://github.com/OpenSTFoundation/openst-contracts/blob/0.10.0-alpha.1/contracts/gnosis_safe_modules/DelayedRecoveryModule.sol) (Allows to replace an owner without Safe confirmations) 
* [CreateAndAddModules](https://github.com/gnosis/safe-contracts/blob/v0.1.0/contracts/libraries/CreateAndAddModules.sol) (Allows to create and add multiple module in one transaction)
* [UserWalletFactory](https://github.com/OpenSTFoundation/openst-contracts/blob/0.10.0-alpha.1/contracts/proxies/UserWalletFactory.sol) (Creates proxy for GnosisSafe, TokenHolder and DelayedRecoveryModule)

```js

// Fetching ABI examples.
let abiBinProvider = new OpenST.AbiBinProvider();
const TokenHolderAbi = abiBinProvider.getABI('TokenHolder');

```

## Deploying contracts

## Constants
Before deploying contracts, please set some constants to funded addresses that you control.

```js

// Initialize web3 object using the geth endpoint
const Web3 = require('web3');
const web3Provider = new Web3('http://127.0.0.1:8545');

// organization owner. Doesn't need to be eth funded.
const organizationOwner = '0xaabb1122....................';

// deployer address
const deployerAddress = '0xaabb1122....................';

// Admin address. Doesn't need to be eth funded.
const adminAddress = '0xaabb1122....................';

// Relayer address
const relayer = '0xaabb1122....................';

// Worker address
const worker = '0xaabb1122....................';

const passphrase = 'some passphrase.....';

// Other constants
const gasPrice = '0x12A05F200';
const gas = 7500000;

```

### Deploy Organization contract

An Organization contract serves as an on-chain access control mechanism by assigning roles to a set of addresses.

```js
// Deploy Organization contract
const Mosaic = require('@openstfoundation/mosaic.js');
const { Organization } = Mosaic.ContractInteract;
const orgConfig = {
  deployer: deployerAddress,
  owner: organizationOwner,
  admin: admin,
  workers: [worker],
  workerExpirationHeight: '20000000' // This is the block height for when the the worker is set to expire.
};
let organizationContractInstance;
let organizationAddress;
Organization.setup(web3Provider, orgConfig).then(function(instance){
  organizationContractInstance = instance;
  organizationAddress = organizationContractInstance.address;
});

```

### Deploy EIP20Token contract

To perform economy setup, you will need an EIP20Token. You can either use an existing EIP20Token contract or deploy a new one.

### Deploy TokenRules contract
One TokenRules contract is deployed per Organization. Only the Organization whitelisted workers can register rule contracts in the TokenRules contract.

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const tokenRulesSetupObject = new OpenST.Setup.TokenRules(web3Provider);
let tokenRulesAddress;
tokenRulesSetupObject.deploy(organization, eip20Token, txOptions).then(function(response){
  tokenRulesAddress = response.receipt.contractAddress;
})

```   

### Deploy TokenHolder MasterCopy

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const userSetup = new OpenST.Setup.User(web3Provider);
let tokenHolderMasterCopyAddress;
userSetup.deployTokenHolderMasterCopy(txOptions).then(function(response){
  tokenHolderMasterCopyAddress = response.receipt.contractAddress;
});

```  

### Deploy Gnosis MasterCopy

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const userSetup = new OpenST.Setup.User(web3Provider);

let gnosisMasterCopyAddress;
userSetup.deployMultiSigMasterCopy(txOptions).then(function(response){
  gnosisMasterCopyAddress = response.receipt.contractAddress;
});

```  

### Deploy DelayedRecoveryModule MasterCopy

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const userSetup = new OpenST.Setup.User(web3Provider);

let delayedRecoveryModuleMasterCopyAddress;
userSetup.deployDelayedRecoveryModuleMasterCopy(txOptions).then(function(response){
  delayedRecoveryModuleMasterCopyAddress = response.receipt.contractAddress;
});

```  

### Deploy CreateAndAddModules Contract

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const userSetup = new OpenST.Setup.User(web3Provider);

let createAndAddModulesAddress;
userSetup.deployCreateAndAddModules(txOptions).then(function(response){
  createAndAddModulesAddress = response.receipt.contractAddress;
});

```  

### Deploy UserWalletFactory Contract

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const userSetup = new OpenST.Setup.User(web3Provider);

let userWalletFactoryAddress;
userSetup.deployUserWalletFactory(txOptions).then(function(response){
  userWalletFactoryAddress = response.receipt.contractAddress;
});

```

### Deploy ProxyFactory Contract

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const userSetup = new OpenST.Setup.User(web3Provider);

let proxyFactoryAddress;
userSetup.deployProxyFactory(txOptions).then(function(response){
  proxyFactoryAddress = response.receipt.contractAddress;
});

``` 

### Deploy PricerRule Contract

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const rulesSetup = new OpenST.Setup.Rules(web3Provider);

const baseCurrencyCode = 'OST';
const conversionRate = 10;
const conversionRateDecimals = 5;
const requiredPriceOracleDecimals = 18;
let pricerRuleAddress;
rulesSetup.deployPricerRule(baseCurrencyCode, conversionRate, conversionRateDecimals, requiredPriceOracleDecimals, txOptions).then(function(response){
  pricerRuleAddress = response.receipt.contractAddress;
})

``` 

## Registration of Rule to TokenRules

```js
txOptions = {
  from: worker,
  gasPrice: gasPrice,
  gas: gas
};

const pricerRule = 'PricerRule';
const tokenRules = new OpenST.Helpers.TokenRules(tokenRulesAddress, web3Provider);
const pricerRuleAbi = abiBinProvider.getABI(pricerRule);
tokenRules.registerRule(pricerRule, pricerRuleAddress, pricerRuleAbi.toString(), txOptions).then(function(response){
  console.log("Successfully registered PricerRule");
})

``` 

## Creating a User Wallet

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const User = new OpenST.Helpers.User(
  tokenHolderMasterCopyAddress, 
  gnosisMasterCopyAddress, 
  delayedRecoveryModuleMasterCopyAddress, 
  createAndAddModulesAddress, 
  eip20Token, 
  tokenRulesAddress,
  userWalletFactoryAddress, 
  proxyFactoryAddress, 
  web3Provider
);
const owner = '0xaabb1122....................';
const ephemeralKey = '0xaabb1122....................';
const sessionKeySpendingLimit = 1000000;
const sessionKeyExpirationHeight = 100000000000; 
const threshold = 1;
const recoveryOwnerAddress = '0xaabb1122....................';
const recoveryControllerAddress = '0xaabb1122....................';
const recoveryBlockDelay = 10;
let gnosisSafeProxy;
let userTokenHolderProxy;
User.createUserWallet(
  [owner], 
  threshold, 
  recoveryOwnerAddress, 
  recoveryControllerAddress, 
  recoveryBlockDelay, 
  [ephemeralKey], 
  [sessionKeySpendingLimit], 
  [sessionKeyExpirationHeight], 
  txOptions
).then(function(response){
  const returnValues = response.events.UserWalletCreated.returnValues;
  const userWalletEvent = JSON.parse(JSON.stringify(returnValues));
  gnosisSafeProxy = userWalletEvent._gnosisSafeProxy;
  userTokenHolderProxy = userWalletEvent._tokenHolderProxy;
});

``` 

## Creating a Company Wallet

```js
txOptions = {
  from: deployerAddress,
  gasPrice: gasPrice,
  gas: gas
};
const User = new OpenST.Helpers.User(
  tokenHolderMasterCopyAddress,
  null, // gnosis safe master copy address
  null, // delayed recovery module master copy address
  null, // create and add modules contract address
  eip20Token,
  tokenRulesAddress,
  userWalletFactoryAddress,
  proxyFactoryAddress,
  web3Provider
);
const owner = '0xaabb1122....................';
const ephemeralKey = '0xaabb1122....................';
const sessionKeySpendingLimit = 1000000;
const sessionKeyExpirationHeight = 100000000000; 
let companyTokenHolderProxy;
User.createCompanyWallet(
  owner,
  [ephemeralKey],
  [sessionKeySpendingLimit],
  [sessionKeyExpirationHeight],
  txOptions
).then(function(response){
  const returnValues = response.events.ProxyCreated.returnValues;
  const proxyEvent = JSON.parse(JSON.stringify(returnValues));
  companyTokenHolderProxy = proxyEvent._proxy;
});

``` 

## Direct transfer of utility tokens

```js
const txOptions = {
  from: relayer,
  gasPrice: gasPrice,
  gas: gas
};
const tokenRules = new OpenST.Helpers.TokenRules(tokenRulesAddress, web3Provider);
const receiver = '0xaabb1122....................'; 
const directTransferCallData = tokenRules.getDirectTransferExecutableData([receiver], [100]);
const nonce = 0; // Ephemeral key current nonce.
const tokenHolder = new OpenST.Helpers.TokenHolder(web3Provider, userTokenHolderProxy);
let transaction = {
  from: userTokenHolderProxy,
  to: tokenRulesAddress,
  data: directTransferCallData,
  nonce: nonce,
  callPrefix: tokenHolder.getTokenHolderExecuteRuleCallPrefix(),
  value: 0,
  gasPrice: 0,
  gas: 0
};
let ephemeralKeyAccountInstance; // Construct account instance of ephemeral key
const vrs = ephemeralKeyAccountInstance.signEIP1077Transaction(transaction);
tokenHolder.executeRule(tokenRulesAddress, directTransferCallData, nonce, vrs.r, vrs.s, vrs.v, txOptions).then(function(response){
  console.log("Successful transfer done!");
})

```

## Setup of PriceOracle 

Make sure PriceOracle contract is already deployed. Refer [PriceOracle](https://github.com/OpenSTFoundation/ost-price-oracle) npm for setup of PriceOracle contract.

## Payment through PricerRule contract 

```js
async function pay() {
  const pricerRule = new OpenST.Helpers.Rules.PricerRule(web3Provider, pricerRuleAddress);
  const workerTxOptions = {
    from: worker,
    gasPrice: gasPrice,
    gas: gas
  };
  let priceOracleAddress; // Set PriceOracle address here.
  const acceptanceMargin = '10000000000'; // set appropriate acceptance margin
  await pricerRule.addPriceOracle(priceOracleAddress, workerTxOptions);
  await pricerRule.setAcceptanceMargin('USD', acceptanceMargin, workerTxOptions);
      
  const txOptions = {
    from: relayer,
    gasPrice: gasPrice,
    gas: gas
  };
  const tokenHolder = new OpenST.Helpers.TokenHolder(web3Provider, userTokenHolderProxy);
  const receiver = '0xaabb1122....................'; 
  const nonce = 0; // Ephemeral key current nonce.
  let baseCurrencyIntendedPrice; // Set current OST/USD price.
  const pricerRulePayCallData = pricerRule.getPayExecutableData(userTokenHolderProxy, [receiver], [1000], 'USD', baseCurrencyIntendedPrice);
  let transaction = {
    from: userTokenHolderProxy,
    to: pricerRuleAddress,
    data: pricerRulePayCallData,
    nonce: nonce,
    callPrefix: tokenHolder.getTokenHolderExecuteRuleCallPrefix(),
    value: 0,
    gasPrice: 0,
    gas: 0
  };
  let ephemeralKeyAccountInstance;
  const signatureObj = ephemeralKeyAccountInstance.signEIP1077Transaction(transaction);
  tokenHolder.executeRule(pricerRuleAddress, pricerRulePayCallData, nonce, signatureObj.r, signatureObj.s, signatureObj.v, txOptions).then( function() {
    console.log("Successful transfer done!");
  })
}

```

## Perform Wallet Operations

### Add Wallet

```js
async function addWallet() {
const txOptions = {
  from: relayer,
  gasPrice: gasPrice,
  gas: gas
};
const gnosisSafe = new GnosisSafe(gnosisSafeProxy, web3Provider);
const threshold = 1;
const ownerToAdd = '0xaabb1122....................';
const ownerToAddWithThresholdCallData = gnosisSafe.getAddOwnerWithThresholdExecutableData(ownerToAdd, threshold);
const nullAddress = '0x0000000000000000000000000000000000000000';
const nonce = await gnosisSafe.getNonce();
const safeTxData = await gnosisSafe.getSafeTxData(gnosisSafeProxy, 0, ownerToAddWithThresholdCallData, 0, 0, 0, 0, nullAddress, nullAddress, nonce);
// 2. Generate EIP712 Signature.
const signatureObj = await owner.signEIP712TypedData(safeTxData);
await gnosisSafe.execTransaction(gnosisSafeProxy, 0, ownerToAddWithThresholdCallData, 0, 0, 0, 0, nullAddress, nullAddress, signatureObj.signature, txOptions);
}
addWallet();    

```

Above similar steps apply for removeWallet and replaceWallet operations.

### Authorize Session

```
async function authorizeSession() {
const txOptions = {
  from: relayer,
  gasPrice: gasPrice,
  gas: gas
};
const tokenHolder = new TokenHolder(web3Provider, userTokenHolderProxy);
const sessionKeyToAuthorize = '0xaabb1122....................';
const spendingLimit = 1000000;
const expirationHeight = 100000000000;
const authorizeSessionCallData = tokenHolderInstance.getAuthorizeSessionExecutableData(sessionKey, spendingLimit, expirationHeight);
const gnosisSafe = new GnosisSafe(gnosisSafeProxy, web3Provider);
const nonce = await gnosisSafe.getNonce();
const safeTxData = gnosisSafeProxyInstance.getSafeTxData(userTokenHolderProxy, 0, authorizeSessionCallData, 0, 0, 0, 0, nullAddress, nullAddress, nonce);
const signatureObj = await owner.signEIP712TypedData(safeTxData);
const result = await gnosisSafe.execTransaction(userTokenHolderProxy, 0, authorizeSessionCallData, 0, 0, 0, 0, nullAddress, nullAddress, signatureObj.signature, txOptions);
}
authorizeSession(); 

```

Above similar above steps apply for revokeSession and logout operations.

## Tests

Tests require docker-compose. To run the tests, execute below command from root directory.

```bash
    // For unit tests.
    npm run test
    // For integration tests.
    npm run test:integration

```   