---
name: voltaire-effect
description: >
  Effect.ts integration for Voltaire with typed errors, services (Provider, Signer),
  and contract patterns. Use for composable blockchain operations with full type safety.
license: MIT
compatibility: Claude Code, Codex, Amp, Cursor
metadata:
  author: TEVM
  version: "1.0"
  docs: https://voltaire-effect.tevm.sh
  github: https://github.com/evmts/voltaire-effect
---

# Voltaire-Effect

Effect.ts integration for Voltaire with typed errors, dependency injection, and composable operations.

## When to Use

- **Use voltaire-effect**: Provider calls, contract interactions, typed errors, composable operations
- **Use voltaire**: Raw encoding/decoding, transaction building without providers

## Installation

```bash
npm install voltaire-effect effect @tevm/voltaire
```

## Core Concept: Typed Effects

Every operation returns `Effect<A, E, R>`:
- `A` - Success value type
- `E` - Typed error union
- `R` - Required services/dependencies

```typescript
import { Effect } from 'effect';
import { Provider, getBalance } from 'voltaire-effect';

// Effect<Uint256, ProviderError | InvalidAddressError, Provider>
const balanceEffect = getBalance(address);

// Run with provider
const result = await Effect.runPromise(
  balanceEffect.pipe(
    Effect.provide(Provider.http("https://eth.llamarpc.com"))
  )
);
```

## Contract Patterns

### Contract Registry (Recommended)

Pre-configure contracts for reuse:

```typescript
import { makeContractRegistry, Contract } from 'voltaire-effect';
import * as Address from '@tevm/voltaire/Address';

// Define contract registry
const contracts = makeContractRegistry({
  usdc: Contract.fromAbi({
    abi: erc20Abi,
    address: Address.from("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
  }),
  weth: Contract.fromAbi({
    abi: wethAbi,
    address: Address.from("0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2")
  })
});

// Use in effects
const getUsdcBalance = (owner: Address) =>
  contracts.usdc.read.balanceOf(owner);

const approveWeth = (spender: Address, amount: Uint256) =>
  contracts.weth.write.approve(spender, amount);
```

### Ad-hoc Contract

For one-off contract interactions:

```typescript
import { Contract } from 'voltaire-effect';

const token = Contract.fromAbi({
  abi: erc20Abi,
  address: tokenAddress
});

// Read call
const balance = token.read.balanceOf(owner);

// Write call (requires Signer)
const transfer = token.write.transfer(recipient, amount);
```

### Factory Pattern (No Address)

For deploying or interacting with multiple instances:

```typescript
const ERC20Factory = Contract.factory({
  abi: erc20Abi
});

// Create instance with address
const usdc = ERC20Factory.attach(usdcAddress);
const dai = ERC20Factory.attach(daiAddress);
```

## Services

### Provider

JSON-RPC provider for read operations:

```typescript
import { Provider } from 'voltaire-effect';

// HTTP provider
const httpProvider = Provider.http("https://eth.llamarpc.com");

// WebSocket provider
const wsProvider = Provider.websocket("wss://eth.llamarpc.com");

// With options
const provider = Provider.http("https://...", {
  timeout: 30000,
  retries: 3
});
```

### Signer

Transaction signing service:

```typescript
import { Signer } from 'voltaire-effect';
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

// From private key
const signer = Signer.fromPrivateKey(
  PrivateKey.from("0x...")
);

// From mnemonic
const signer = Signer.fromMnemonic(
  "word1 word2 ... word12",
  "m/44'/60'/0'/0/0"
);
```

### ContractRegistryService

Dependency injection for contracts:

```typescript
import { ContractRegistryService, makeContractRegistry } from 'voltaire-effect';
import { Effect, Layer } from 'effect';

const registry = makeContractRegistry({ usdc, weth, uniswap });

// Create layer
const ContractLayer = Layer.succeed(
  ContractRegistryService,
  registry
);

// Use in effects
const swap = Effect.gen(function* () {
  const contracts = yield* ContractRegistryService;
  yield* contracts.usdc.write.approve(uniswapAddress, amount);
  yield* contracts.uniswap.write.swap(params);
});
```

## Typed Errors

All errors are typed and can be pattern matched:

```typescript
import { Effect } from 'effect';
import {
  ProviderError,
  ContractError,
  InsufficientFundsError,
  RevertError
} from 'voltaire-effect';

const safeTransfer = transfer(recipient, amount).pipe(
  Effect.catchTag("InsufficientFundsError", (e) =>
    Effect.fail(new UserFriendlyError(`Not enough balance: ${e.required}`))
  ),
  Effect.catchTag("RevertError", (e) =>
    Effect.fail(new UserFriendlyError(`Contract reverted: ${e.reason}`))
  ),
  Effect.catchTag("ProviderError", (e) =>
    Effect.retry({ times: 3, schedule: Schedule.exponential(1000) })
  )
);
```

## Composability

### Retry Logic

```typescript
import { Effect, Schedule } from 'effect';

const robustCall = getBalance(address).pipe(
  Effect.retry({
    times: 5,
    schedule: Schedule.exponential("100 millis")
  })
);
```

### Timeout

```typescript
const timedCall = getBalance(address).pipe(
  Effect.timeout("10 seconds")
);
```

### Parallel Execution

```typescript
const [balance1, balance2, balance3] = await Effect.runPromise(
  Effect.all([
    getBalance(addr1),
    getBalance(addr2),
    getBalance(addr3)
  ], { concurrency: 3 })
);
```

### Sequential with Dependencies

```typescript
const approveAndTransfer = Effect.gen(function* () {
  yield* token.write.approve(spender, amount);
  yield* token.write.transferFrom(owner, recipient, amount);
});
```

## Complete Example

```typescript
import { Effect, Layer } from 'effect';
import {
  Provider,
  Signer,
  makeContractRegistry,
  Contract
} from 'voltaire-effect';
import * as Address from '@tevm/voltaire/Address';
import * as Uint256 from '@tevm/voltaire/Uint256';

// Setup contracts
const contracts = makeContractRegistry({
  usdc: Contract.fromAbi({
    abi: erc20Abi,
    address: Address.from("0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48")
  })
});

// Define operation
const transferUsdc = (to: Address, amount: Uint256) =>
  Effect.gen(function* () {
    const balance = yield* contracts.usdc.read.balanceOf(myAddress);

    if (Uint256.lt(balance, amount)) {
      return yield* Effect.fail(new InsufficientFundsError({ required: amount, available: balance }));
    }

    const txHash = yield* contracts.usdc.write.transfer(to, amount);
    return txHash;
  });

// Setup layers
const MainLayer = Layer.mergeAll(
  Provider.http("https://eth.llamarpc.com"),
  Signer.fromPrivateKey(privateKey)
);

// Run
const result = await Effect.runPromise(
  transferUsdc(recipient, amount).pipe(
    Effect.provide(MainLayer)
  )
);
```

## Reference Docs

See [references/SERVICES.md](./references/SERVICES.md) for service patterns.
See [references/CONTRACT.md](./references/CONTRACT.md) for contract patterns.

## Links

- [Documentation](https://voltaire-effect.tevm.sh)
- [Contract Registry](https://voltaire-effect.tevm.sh/services/contract-registry)
- [Provider](https://voltaire-effect.tevm.sh/services/provider)
- [Signer](https://voltaire-effect.tevm.sh/services/signer)
- [Typed Errors](https://voltaire-effect.tevm.sh/errors)
- [Effect.ts](https://effect.website)

## Related

- Use [voltaire](../voltaire/SKILL.md) for low-level primitives without Effect.ts
