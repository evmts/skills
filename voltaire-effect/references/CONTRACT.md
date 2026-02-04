# Voltaire-Effect Contract Patterns

Complete reference for contract interaction patterns.

## Contract Creation

### From ABI with Address (Instance)

Most common pattern - a specific contract instance:

```typescript
import { Contract } from 'voltaire-effect';
import * as Address from '@tevm/voltaire/Address';

const usdc = Contract.fromAbi({
  abi: erc20Abi,
  address: Address.from("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
});

// Read methods
usdc.read.name();           // Effect<string, ContractError, Provider>
usdc.read.symbol();         // Effect<string, ContractError, Provider>
usdc.read.balanceOf(owner); // Effect<Uint256, ContractError, Provider>
usdc.read.allowance(owner, spender);

// Write methods
usdc.write.transfer(to, amount);  // Effect<TxHash, ContractError, Provider | Signer>
usdc.write.approve(spender, amount);
usdc.write.transferFrom(from, to, amount);
```

### Factory Pattern (No Address)

For deploying or working with multiple instances:

```typescript
const ERC20 = Contract.factory({
  abi: erc20Abi
});

// Attach to specific address
const usdc = ERC20.attach(usdcAddress);
const dai = ERC20.attach(daiAddress);

// Deploy new instance
const deployTx = ERC20.deploy({
  bytecode: erc20Bytecode,
  args: ["My Token", "MTK", 18]
});
```

### From Human-Readable ABI

```typescript
const token = Contract.fromHumanReadable({
  abi: [
    "function name() view returns (string)",
    "function symbol() view returns (string)",
    "function balanceOf(address owner) view returns (uint256)",
    "function transfer(address to, uint256 amount) returns (bool)",
    "event Transfer(address indexed from, address indexed to, uint256 amount)"
  ],
  address: tokenAddress
});
```

### From Etherscan/Sourcify

```typescript
// Auto-fetch ABI from Etherscan
const contract = await Contract.fromEtherscan({
  address: contractAddress,
  apiKey: etherscanApiKey
});

// From Sourcify
const contract = await Contract.fromSourcify({
  address: contractAddress,
  chainId: 1
});
```

## Read Operations

Read operations require only `Provider`:

```typescript
import { Effect } from 'effect';
import { Provider } from 'voltaire-effect';

// Simple read
const name = usdc.read.name();

// Read with args
const balance = usdc.read.balanceOf(ownerAddress);

// Read with block tag
const historicalBalance = usdc.read.balanceOf(ownerAddress, {
  blockTag: 15000000n
});

// Multiple reads in parallel
const [name, symbol, decimals, totalSupply] = await Effect.runPromise(
  Effect.all([
    usdc.read.name(),
    usdc.read.symbol(),
    usdc.read.decimals(),
    usdc.read.totalSupply()
  ], { concurrency: 4 }).pipe(
    Effect.provide(Provider.http("https://..."))
  )
);
```

## Write Operations

Write operations require `Provider` and `Signer`:

```typescript
import { Effect, Layer } from 'effect';
import { Provider, Signer } from 'voltaire-effect';

// Simple write
const txHash = usdc.write.transfer(recipient, amount);

// Write with options
const txHash = usdc.write.transfer(recipient, amount, {
  gasLimit: 100000n,
  maxFeePerGas: Uint256.from(30000000000n),
  maxPriorityFeePerGas: Uint256.from(1000000000n),
  nonce: 5n
});

// Execute
const AppLayer = Layer.mergeAll(
  Provider.http("https://..."),
  Signer.fromPrivateKey(privateKey)
);

const hash = await Effect.runPromise(
  usdc.write.transfer(recipient, amount).pipe(
    Effect.provide(AppLayer)
  )
);
```

## Event Handling

### Decoding Events

```typescript
// Decode a log
const decoded = usdc.decodeLog(log);
// { eventName: 'Transfer', args: { from, to, amount } }

// Type-safe event access
if (decoded.eventName === 'Transfer') {
  console.log(decoded.args.from);   // Address
  console.log(decoded.args.to);     // Address
  console.log(decoded.args.amount); // Uint256
}
```

### Filtering Events

```typescript
import { getLogs } from 'voltaire-effect';

// Get Transfer events
const transferLogs = getLogs({
  address: usdc.address,
  topics: usdc.encodeEventTopics('Transfer', {
    from: senderAddress  // Filter by sender
  }),
  fromBlock: 18000000n,
  toBlock: 'latest'
});

// Decode all logs
const transfers = Effect.gen(function* () {
  const logs = yield* transferLogs;
  return logs.map(log => usdc.decodeLog(log));
});
```

### Event Subscriptions (WebSocket)

```typescript
import { Provider, subscribe } from 'voltaire-effect';

const subscription = subscribe({
  address: usdc.address,
  topics: usdc.encodeEventTopics('Transfer')
}).pipe(
  Effect.tap(log => {
    const event = usdc.decodeLog(log);
    console.log(`Transfer: ${event.args.from} -> ${event.args.to}`);
  }),
  Effect.provide(Provider.websocket("wss://..."))
);
```

## Contract Registry

### Basic Registry

```typescript
import { makeContractRegistry, Contract } from 'voltaire-effect';

const contracts = makeContractRegistry({
  usdc: Contract.fromAbi({ abi: erc20Abi, address: usdcAddress }),
  weth: Contract.fromAbi({ abi: wethAbi, address: wethAddress }),
  uniswap: Contract.fromAbi({ abi: routerAbi, address: routerAddress })
});

// Type-safe access to all contracts
contracts.usdc.read.balanceOf(owner);
contracts.weth.write.deposit({ value: amount });
contracts.uniswap.write.swapExactTokensForTokens(...);
```

### Registry with Factories

```typescript
const contracts = makeContractRegistry({
  // Instances (with address)
  usdc: Contract.fromAbi({ abi: erc20Abi, address: usdcAddress }),

  // Factories (without address)
  erc20: Contract.factory({ abi: erc20Abi }),
  erc721: Contract.factory({ abi: erc721Abi })
});

// Use factory to create instances
const dai = contracts.erc20.attach(daiAddress);
const nft = contracts.erc721.attach(nftAddress);
```

### Multi-Chain Registry

```typescript
const mainnetContracts = makeContractRegistry({
  usdc: Contract.fromAbi({
    abi: erc20Abi,
    address: Address.from("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
  })
});

const arbitrumContracts = makeContractRegistry({
  usdc: Contract.fromAbi({
    abi: erc20Abi,
    address: Address.from("0xaf88d065e77c8cC2239327C5EDb3A432268e5831")
  })
});

// Use with appropriate provider
const mainnetBalance = mainnetContracts.usdc.read.balanceOf(owner).pipe(
  Effect.provide(Provider.http("https://eth.llamarpc.com"))
);

const arbitrumBalance = arbitrumContracts.usdc.read.balanceOf(owner).pipe(
  Effect.provide(Provider.http("https://arb.llamarpc.com"))
);
```

## Error Handling

### Contract-Specific Errors

```typescript
import { Effect } from 'effect';
import { RevertError, ContractError } from 'voltaire-effect';

const safeTransfer = usdc.write.transfer(to, amount).pipe(
  Effect.catchTag("RevertError", (error) => {
    // error.reason contains revert reason
    // error.data contains raw revert data
    console.log(`Reverted: ${error.reason}`);
    return Effect.fail(new UserError("Transfer failed"));
  }),
  Effect.catchTag("ContractError", (error) => {
    // Generic contract error
    return Effect.fail(new UserError("Contract call failed"));
  })
);
```

### Custom Error Decoding

```typescript
// Define custom errors in ABI
const abi = [
  "error InsufficientBalance(uint256 available, uint256 required)",
  "error Unauthorized(address caller)"
];

const contract = Contract.fromHumanReadable({ abi, address });

// Errors are automatically decoded
const transfer = contract.write.transfer(to, amount).pipe(
  Effect.catchTag("RevertError", (error) => {
    if (error.errorName === "InsufficientBalance") {
      const { available, required } = error.args;
      return Effect.fail(new UserError(
        `Need ${required} but only have ${available}`
      ));
    }
    if (error.errorName === "Unauthorized") {
      return Effect.fail(new UserError("Not authorized"));
    }
    return Effect.fail(error);
  })
);
```

## Gas Estimation

```typescript
// Estimate gas for a write operation
const gasEstimate = usdc.estimateGas.transfer(to, amount);

// Use estimate in transaction
const transfer = Effect.gen(function* () {
  const gas = yield* usdc.estimateGas.transfer(to, amount);
  const gasWithBuffer = Uint256.mul(gas, Uint256.from(120n)) / 100n; // +20%

  return yield* usdc.write.transfer(to, amount, {
    gasLimit: gasWithBuffer
  });
});
```

## Simulation

```typescript
// Simulate without sending
const result = usdc.simulate.transfer(to, amount);
// Returns: Effect<{ result, gasUsed, logs }, SimulationError, Provider>

// Check if call would succeed
const wouldSucceed = Effect.gen(function* () {
  const sim = yield* usdc.simulate.transfer(to, amount);
  return sim.result; // true/false for ERC20 transfer
});
```

## Multicall

```typescript
import { multicall } from 'voltaire-effect';

// Batch multiple reads into single RPC call
const results = multicall([
  usdc.read.balanceOf(addr1),
  usdc.read.balanceOf(addr2),
  usdc.read.balanceOf(addr3),
  weth.read.balanceOf(addr1)
]);
// Single RPC call, returns array of results
```

## Access List Generation

```typescript
// Generate access list for transaction
const accessList = usdc.createAccessList.transfer(to, amount);

// Use in transaction
const transfer = Effect.gen(function* () {
  const list = yield* usdc.createAccessList.transfer(to, amount);
  return yield* usdc.write.transfer(to, amount, {
    accessList: list
  });
});
```
