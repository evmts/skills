# Voltaire-Effect Services Reference

Complete reference for all voltaire-effect services.

## Provider Service

JSON-RPC provider for blockchain read operations.

### Creating Providers

```typescript
import { Provider } from 'voltaire-effect';

// HTTP provider (most common)
const httpProvider = Provider.http("https://eth.llamarpc.com");

// WebSocket provider (for subscriptions)
const wsProvider = Provider.websocket("wss://eth.llamarpc.com");

// With configuration
const configuredProvider = Provider.http("https://...", {
  timeout: 30000,      // Request timeout in ms
  retries: 3,          // Auto-retry count
  retryDelay: 1000,    // Delay between retries
  batchSize: 100,      // Max batch request size
  headers: {           // Custom headers
    "Authorization": "Bearer ..."
  }
});

// IPC provider (local node)
const ipcProvider = Provider.ipc("/path/to/geth.ipc");
```

### Provider Methods

```typescript
import { Effect } from 'effect';
import {
  getBalance,
  getBlockNumber,
  getBlock,
  getTransaction,
  getTransactionReceipt,
  call,
  estimateGas,
  getCode,
  getStorageAt,
  getLogs,
  getGasPrice,
  getMaxPriorityFeePerGas
} from 'voltaire-effect';

// All methods return Effect<A, ProviderError, Provider>

// Account queries
const balance = getBalance(address);
const nonce = getTransactionCount(address);
const code = getCode(address);

// Block queries
const blockNumber = getBlockNumber();
const block = getBlock(blockNumber);
const blockWithTxs = getBlock(blockNumber, { includeTransactions: true });

// Transaction queries
const tx = getTransaction(txHash);
const receipt = getTransactionReceipt(txHash);

// Contract interaction
const result = call({
  to: contractAddress,
  data: calldata
});

const gasEstimate = estimateGas({
  from: sender,
  to: recipient,
  value: amount
});

// Storage
const slot = getStorageAt(address, Bytes32.from("0x0"));

// Logs
const logs = getLogs({
  address: contractAddress,
  topics: [transferTopic],
  fromBlock: 1000000n,
  toBlock: 'latest'
});

// Gas prices
const gasPrice = getGasPrice();
const priorityFee = getMaxPriorityFeePerGas();
```

### Using Provider in Effects

```typescript
import { Effect } from 'effect';
import { Provider, getBalance, getBlockNumber } from 'voltaire-effect';

const myEffect = Effect.gen(function* () {
  const balance = yield* getBalance(myAddress);
  const block = yield* getBlockNumber();
  return { balance, block };
});

// Provide the provider
const result = await Effect.runPromise(
  myEffect.pipe(
    Effect.provide(Provider.http("https://eth.llamarpc.com"))
  )
);
```

## Signer Service

Transaction signing service for write operations.

### Creating Signers

```typescript
import { Signer } from 'voltaire-effect';
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

// From private key
const pkSigner = Signer.fromPrivateKey(
  PrivateKey.from("0x...")
);

// From mnemonic (BIP-39)
const mnemonicSigner = Signer.fromMnemonic(
  "word1 word2 ... word12",
  "m/44'/60'/0'/0/0"  // derivation path
);

// From mnemonic with passphrase
const passphraseSigner = Signer.fromMnemonic(
  "word1 word2 ... word12",
  "m/44'/60'/0'/0/0",
  "my-passphrase"
);

// HD wallet with multiple accounts
const hdSigner = Signer.hd({
  mnemonic: "word1 word2 ... word12",
  basePath: "m/44'/60'/0'/0",
  accounts: 10  // derive 10 accounts
});
```

### Signer Methods

```typescript
import {
  sendTransaction,
  signMessage,
  signTypedData,
  getAddress
} from 'voltaire-effect';

// Get signer address
const address = getAddress();

// Send transaction
const txHash = sendTransaction({
  to: recipient,
  value: amount,
  gasLimit: 21000n
});

// Sign message (EIP-191)
const signature = signMessage("Hello World");

// Sign typed data (EIP-712)
const typedSig = signTypedData({
  domain: { name: "MyApp", version: "1", chainId: 1n },
  types: { Order: [...] },
  message: orderData
});
```

### Using Signer in Effects

```typescript
import { Effect, Layer } from 'effect';
import { Provider, Signer, sendTransaction } from 'voltaire-effect';

const sendEth = Effect.gen(function* () {
  const txHash = yield* sendTransaction({
    to: recipient,
    value: Uint256.from(1000000000000000000n) // 1 ETH
  });
  return txHash;
});

// Provide both provider and signer
const AppLayer = Layer.mergeAll(
  Provider.http("https://eth.llamarpc.com"),
  Signer.fromPrivateKey(privateKey)
);

const result = await Effect.runPromise(
  sendEth.pipe(Effect.provide(AppLayer))
);
```

## ContractRegistryService

Dependency injection for pre-configured contracts.

### Creating a Registry

```typescript
import { makeContractRegistry, Contract } from 'voltaire-effect';

const registry = makeContractRegistry({
  // Each key becomes a property on the registry
  usdc: Contract.fromAbi({
    abi: erc20Abi,
    address: usdcAddress
  }),
  weth: Contract.fromAbi({
    abi: wethAbi,
    address: wethAddress
  }),
  uniswapRouter: Contract.fromAbi({
    abi: uniswapRouterAbi,
    address: routerAddress
  })
});

// Type-safe access
registry.usdc.read.balanceOf(owner);
registry.weth.write.deposit();
registry.uniswapRouter.write.swapExactTokensForTokens(...);
```

### Using as a Service

```typescript
import { Effect, Layer, Context } from 'effect';
import { ContractRegistryService } from 'voltaire-effect';

// Define the registry type
type MyRegistry = typeof registry;

// Create the service tag
const MyContracts = Context.GenericTag<MyRegistry>("MyContracts");

// Create layer
const ContractsLayer = Layer.succeed(MyContracts, registry);

// Use in effects
const swap = Effect.gen(function* () {
  const contracts = yield* MyContracts;

  // Approve USDC
  yield* contracts.usdc.write.approve(
    contracts.uniswapRouter.address,
    amount
  );

  // Execute swap
  const txHash = yield* contracts.uniswapRouter.write.swapExactTokensForTokens(
    amountIn,
    amountOutMin,
    path,
    recipient,
    deadline
  );

  return txHash;
});

// Run with layer
await Effect.runPromise(
  swap.pipe(
    Effect.provide(ContractsLayer),
    Effect.provide(Provider.http("...")),
    Effect.provide(Signer.fromPrivateKey(pk))
  )
);
```

## KeccakService

Hashing service (typically provided automatically).

```typescript
import { KeccakService, keccak256 } from 'voltaire-effect';

const hash = keccak256(data);
// Effect<Keccak256Hash, never, KeccakService>
```

## Secp256k1Service

Cryptographic operations service.

```typescript
import { Secp256k1Service, sign, verify, recover } from 'voltaire-effect';

const signature = sign(messageHash, privateKey);
const isValid = verify(messageHash, signature, publicKey);
const recoveredPubKey = recover(messageHash, signature);
```

## RlpService

RLP encoding/decoding service.

```typescript
import { RlpService, rlpEncode, rlpDecode } from 'voltaire-effect';

const encoded = rlpEncode(data);
const decoded = rlpDecode(encoded);
```

## Combining Services

```typescript
import { Effect, Layer } from 'effect';
import { Provider, Signer, makeContractRegistry } from 'voltaire-effect';

// Build the full application layer
const AppLayer = Layer.mergeAll(
  // Provider for reads
  Provider.http("https://eth.llamarpc.com"),

  // Signer for writes
  Signer.fromPrivateKey(privateKey),

  // Contracts
  Layer.succeed(MyContracts, registry)
);

// All effects can now access all services
const complexOperation = Effect.gen(function* () {
  const contracts = yield* MyContracts;
  const balance = yield* contracts.usdc.read.balanceOf(myAddress);
  // ... more operations
}).pipe(Effect.provide(AppLayer));
```

## Service Scoping

```typescript
import { Effect, Layer, Scope } from 'effect';

// Scoped provider (auto-cleanup)
const ScopedProvider = Provider.http("...", {
  // Will be cleaned up when scope ends
});

// Use with scope
const program = Effect.scoped(
  Effect.gen(function* () {
    // Provider is available here
    const balance = yield* getBalance(address);
    return balance;
  }).pipe(Effect.provide(ScopedProvider))
);
```
