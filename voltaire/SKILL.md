---
name: voltaire
description: >
  Voltaire Ethereum primitives library. Use for branded types (Address, Uint256, Hex),
  ABI encoding, RLP serialization, transaction building, and cryptographic operations.
  Does NOT include provider/signer abstractions - use voltaire-effect for that.
license: MIT
compatibility: Claude Code, Codex, Amp, Cursor
metadata:
  author: TEVM
  version: "1.0"
  docs: https://voltaire.tevm.sh
  github: https://github.com/evmts/voltaire
---

# Voltaire

Low-level Ethereum primitives library with branded types and zero provider/signer abstractions.

## When to Use

- **Use Voltaire**: Encoding, decoding, type validation, transaction building, signatures
- **Use voltaire-effect**: Provider calls, contract interactions, typed errors with Effect.ts

## Installation

```bash
npm install @tevm/voltaire
# or
pnpm add @tevm/voltaire
```

## Core Concept: Branded Types

Voltaire uses branded `Uint8Array` types for type safety at compile time:

```typescript
// WRONG - raw strings are not type-safe
const addr = "0x1234...";

// CORRECT - branded types validate at construction
import * as Address from '@tevm/voltaire/Address';
const addr = Address.from("0x1234..."); // Returns Address branded type
```

## Namespace Pattern

Import primitives as namespaces for tree-shaking:

```typescript
import * as Address from '@tevm/voltaire/Address';
import * as Uint256 from '@tevm/voltaire/Uint256';
import * as Hex from '@tevm/voltaire/Hex';
import * as Abi from '@tevm/voltaire/Abi';
import * as Rlp from '@tevm/voltaire/Rlp';
import * as Transaction from '@tevm/voltaire/Transaction';
```

## Key Primitives

### Address
```typescript
import * as Address from '@tevm/voltaire/Address';

const addr = Address.from("0x1234567890123456789012345678901234567890");
Address.toHex(addr);           // "0x1234..."
Address.toChecksum(addr);      // "0x1234..." with EIP-55 checksum
Address.isValid("0x...");      // boolean validation
Address.equals(addr1, addr2);  // constant-time comparison
```

### Uint256 / Numeric Types
```typescript
import * as Uint256 from '@tevm/voltaire/Uint256';
import * as Uint8 from '@tevm/voltaire/Uint8';

const value = Uint256.from(1000000000000000000n); // 1 ETH in wei
Uint256.toHex(value);
Uint256.toBigInt(value);
Uint256.add(a, b);
Uint256.mul(a, b);
```

### Hex
```typescript
import * as Hex from '@tevm/voltaire/Hex';

const hex = Hex.from("0xdeadbeef");
Hex.toBytes(hex);
Hex.fromBytes(bytes);
Hex.concat(hex1, hex2);
Hex.slice(hex, 0, 4);
```

### ABI Encoding
```typescript
import * as Abi from '@tevm/voltaire/Abi';

// Encode function call
const data = Abi.encodeCall({
  abi: contractAbi,
  functionName: 'transfer',
  args: [recipient, amount]
});

// Decode return value
const result = Abi.decodeResult({
  abi: contractAbi,
  functionName: 'balanceOf',
  data: returnData
});

// Encode/decode parameters
Abi.encodeParameters(['address', 'uint256'], [addr, amount]);
Abi.decodeParameters(['address', 'uint256'], data);
```

### RLP Encoding
```typescript
import * as Rlp from '@tevm/voltaire/Rlp';

const encoded = Rlp.encode([addr, nonce, gasPrice]);
const decoded = Rlp.decode(encoded);
```

### Transaction Building
```typescript
import * as Transaction from '@tevm/voltaire/Transaction';

// EIP-1559 transaction
const tx = Transaction.from({
  type: 'eip1559',
  chainId: 1n,
  nonce: 0n,
  to: recipient,
  value: Uint256.from(1000000000000000000n),
  maxFeePerGas: Uint256.from(30000000000n),
  maxPriorityFeePerGas: Uint256.from(1000000000n),
  gasLimit: 21000n,
  data: Hex.from("0x")
});

// Serialize for signing
const serialized = Transaction.serialize(tx);

// Sign transaction
const signed = Transaction.sign(tx, privateKey);
```

### Signatures
```typescript
import * as Signature from '@tevm/voltaire/Signature';
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

const privateKey = PrivateKey.from("0x...");
const signature = Signature.sign(messageHash, privateKey);

Signature.recover(messageHash, signature); // Returns PublicKey
Signature.toHex(signature);                // "0x..." (65 bytes)
```

### Access Lists (EIP-2930)
```typescript
import * as AccessList from '@tevm/voltaire/AccessList';

const accessList = AccessList.from([
  {
    address: contractAddr,
    storageKeys: [slot0, slot1]
  }
]);
```

### Authorizations (EIP-7702)
```typescript
import * as Authorization from '@tevm/voltaire/Authorization';

const auth = Authorization.from({
  chainId: 1n,
  address: implementationAddr,
  nonce: 0n
});
```

## Full Primitive List

See [references/PRIMITIVES.md](./references/PRIMITIVES.md) for complete list of 140+ primitives.

## Common Patterns

See [references/PATTERNS.md](./references/PATTERNS.md) for code patterns.

## Links

- [Documentation](https://voltaire.tevm.sh)
- [Getting Started](https://voltaire.tevm.sh/getting-started)
- [Primitives Reference](https://voltaire.tevm.sh/primitives)
- [Crypto Reference](https://voltaire.tevm.sh/crypto)
- [GitHub](https://github.com/evmts/voltaire)
- [Playground](https://playground.tevm.sh)

## Related

- Use [voltaire-effect](../voltaire-effect/SKILL.md) for provider/signer abstractions with Effect.ts
