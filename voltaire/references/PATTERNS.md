# Voltaire Code Patterns

Common patterns for using Voltaire primitives effectively.

## Pattern 1: Always Use Constructors

Never pass raw strings where branded types are expected:

```typescript
// WRONG - bypasses validation
function transfer(to: string, amount: bigint) {
  // No validation, potential bugs
}

// CORRECT - use branded types
import * as Address from '@tevm/voltaire/Address';
import * as Uint256 from '@tevm/voltaire/Uint256';

function transfer(to: Address, amount: Uint256) {
  // Types guarantee valid values
}

// At call site
transfer(
  Address.from("0x1234..."),
  Uint256.from(1000000000000000000n)
);
```

## Pattern 2: Early Validation

Validate user input at system boundaries:

```typescript
import * as Address from '@tevm/voltaire/Address';

function handleUserInput(addressInput: string) {
  // Validate early
  if (!Address.isValid(addressInput)) {
    throw new Error('Invalid address');
  }

  const addr = Address.from(addressInput);
  // Rest of function uses branded type
  return processAddress(addr);
}
```

## Pattern 3: Namespace Imports for Tree-Shaking

Import as namespaces, not destructured:

```typescript
// WRONG - may break tree-shaking
import { from, toHex } from '@tevm/voltaire/Address';

// CORRECT - tree-shakable
import * as Address from '@tevm/voltaire/Address';

Address.from("0x...");
Address.toHex(addr);
```

## Pattern 4: Type Narrowing with Generics

```typescript
import * as Transaction from '@tevm/voltaire/Transaction';

function processTransaction<T extends Transaction>(tx: T) {
  const type = Transaction.getType(tx);

  switch (type) {
    case 'eip1559':
      // tx is narrowed to EIP1559Transaction
      return handleEip1559(tx);
    case 'eip4844':
      // tx is narrowed to EIP4844Transaction
      return handleBlob(tx);
    default:
      return handleLegacy(tx);
  }
}
```

## Pattern 5: Building EIP-1559 Transactions

```typescript
import * as Transaction from '@tevm/voltaire/Transaction';
import * as Address from '@tevm/voltaire/Address';
import * as Uint256 from '@tevm/voltaire/Uint256';
import * as Hex from '@tevm/voltaire/Hex';
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

const tx = Transaction.from({
  type: 'eip1559',
  chainId: 1n,
  nonce: 5n,
  to: Address.from("0x..."),
  value: Uint256.from(1000000000000000000n), // 1 ETH
  maxFeePerGas: Uint256.from(30000000000n),   // 30 gwei
  maxPriorityFeePerGas: Uint256.from(1000000000n), // 1 gwei
  gasLimit: 21000n,
  data: Hex.from("0x")
});

const privateKey = PrivateKey.from("0x...");
const signed = Transaction.sign(tx, privateKey);
const serialized = Transaction.serialize(signed);
```

## Pattern 6: ABI Encoding Contract Calls

```typescript
import * as Abi from '@tevm/voltaire/Abi';
import * as Address from '@tevm/voltaire/Address';
import * as Uint256 from '@tevm/voltaire/Uint256';

const erc20Abi = [
  {
    type: 'function',
    name: 'transfer',
    inputs: [
      { name: 'to', type: 'address' },
      { name: 'amount', type: 'uint256' }
    ],
    outputs: [{ type: 'bool' }]
  }
] as const;

// Encode function call
const calldata = Abi.encodeCall({
  abi: erc20Abi,
  functionName: 'transfer',
  args: [
    Address.from("0x..."),
    Uint256.from(1000000n)
  ]
});

// Decode return value
const success = Abi.decodeResult({
  abi: erc20Abi,
  functionName: 'transfer',
  data: returnData
});
```

## Pattern 7: Event Log Decoding

```typescript
import * as Abi from '@tevm/voltaire/Abi';
import * as EventLog from '@tevm/voltaire/EventLog';

const transferEvent = {
  type: 'event',
  name: 'Transfer',
  inputs: [
    { name: 'from', type: 'address', indexed: true },
    { name: 'to', type: 'address', indexed: true },
    { name: 'amount', type: 'uint256', indexed: false }
  ]
} as const;

function decodeTransferLog(log: EventLog) {
  const decoded = Abi.decodeEventLog({
    abi: [transferEvent],
    data: log.data,
    topics: log.topics
  });

  return {
    from: decoded.from,   // Address
    to: decoded.to,       // Address
    amount: decoded.amount // Uint256
  };
}
```

## Pattern 8: EIP-712 Typed Data Signing

```typescript
import * as TypedData from '@tevm/voltaire/TypedData';
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

const domain = {
  name: 'MyApp',
  version: '1',
  chainId: 1n,
  verifyingContract: Address.from("0x...")
};

const types = {
  Order: [
    { name: 'maker', type: 'address' },
    { name: 'amount', type: 'uint256' },
    { name: 'nonce', type: 'uint256' }
  ]
};

const message = {
  maker: Address.from("0x..."),
  amount: Uint256.from(1000n),
  nonce: Uint256.from(1n)
};

const privateKey = PrivateKey.from("0x...");
const signature = TypedData.sign(domain, types, message, privateKey);
```

## Pattern 9: Access List Construction

```typescript
import * as AccessList from '@tevm/voltaire/AccessList';
import * as Address from '@tevm/voltaire/Address';
import * as Bytes32 from '@tevm/voltaire/Bytes32';

const accessList = AccessList.from([
  {
    address: Address.from("0x...contract..."),
    storageKeys: [
      Bytes32.from("0x0000...0000"), // slot 0
      Bytes32.from("0x0000...0001"), // slot 1
    ]
  },
  {
    address: Address.from("0x...token..."),
    storageKeys: []
  }
]);
```

## Pattern 10: Signature Recovery

```typescript
import * as Signature from '@tevm/voltaire/Signature';
import * as Keccak256 from '@tevm/voltaire/Keccak256';
import * as Address from '@tevm/voltaire/Address';

function verifyMessage(message: string, signature: Signature, expectedSigner: Address): boolean {
  // Hash the message with Ethereum prefix
  const prefix = `\x19Ethereum Signed Message:\n${message.length}`;
  const messageHash = Keccak256.hashText(prefix + message);

  // Recover public key from signature
  const publicKey = Signature.recover(messageHash, signature);

  // Derive address from public key
  const recoveredAddress = Address.fromPublicKey(publicKey);

  // Compare addresses
  return Address.equals(recoveredAddress, expectedSigner);
}
```

## Pattern 11: Denomination Conversion

```typescript
import * as Wei from '@tevm/voltaire/Wei';
import * as Gwei from '@tevm/voltaire/Gwei';
import * as Ether from '@tevm/voltaire/Ether';

// User input in ETH
const userAmount = "1.5"; // ETH

// Convert to wei for transaction
const weiAmount = Ether.toWei(userAmount);

// Gas price in gwei
const gasPriceGwei = "30";
const gasPriceWei = Gwei.toWei(gasPriceGwei);

// Format wei for display
const displayAmount = Ether.format(weiAmount, 4); // "1.5000"
```

## Pattern 12: HD Wallet Derivation

```typescript
import * as Bip39 from '@tevm/voltaire/Bip39';
import * as HdWallet from '@tevm/voltaire/HdWallet';
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

// Generate mnemonic
const mnemonic = Bip39.generate(12);

// Derive seed
const seed = Bip39.toSeed(mnemonic);

// Derive Ethereum account (BIP-44 path)
const path = "m/44'/60'/0'/0/0";
const hdKey = HdWallet.derive(seed, path);
const privateKey = PrivateKey.from(hdKey.privateKey);
const address = PrivateKey.toAddress(privateKey);
```

## Anti-Patterns to Avoid

### Don't Mix Raw and Branded Types

```typescript
// WRONG
function process(addr: Address | string) {
  // Type confusion
}

// CORRECT
function process(addr: Address) {
  // Always branded
}
```

### Don't Skip Validation

```typescript
// WRONG - unsafe cast
const addr = "0x123" as unknown as Address;

// CORRECT - validate
const addr = Address.from("0x123...");
```

### Don't Mutate Branded Types

```typescript
// WRONG - branded types are immutable
addr[0] = 0xff;

// CORRECT - create new value
const newAddr = Address.from(modifiedBytes);
```
