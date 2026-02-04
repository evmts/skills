# Voltaire Primitives Reference

Complete list of all Voltaire primitives organized by category.

## Numeric Types

| Type | Description | Range |
|------|-------------|-------|
| `Uint8` | 8-bit unsigned | 0 to 255 |
| `Uint16` | 16-bit unsigned | 0 to 65,535 |
| `Uint32` | 32-bit unsigned | 0 to 4,294,967,295 |
| `Uint64` | 64-bit unsigned | 0 to 18,446,744,073,709,551,615 |
| `Uint128` | 128-bit unsigned | 0 to 2^128-1 |
| `Uint256` | 256-bit unsigned | 0 to 2^256-1 |
| `Int8` | 8-bit signed | -128 to 127 |
| `Int16` | 16-bit signed | -32,768 to 32,767 |
| `Int32` | 32-bit signed | -2^31 to 2^31-1 |
| `Int64` | 64-bit signed | -2^63 to 2^63-1 |
| `Int128` | 128-bit signed | -2^127 to 2^127-1 |
| `Int256` | 256-bit signed | -2^255 to 2^255-1 |

```typescript
import * as Uint256 from '@tevm/voltaire/Uint256';

Uint256.from(bigint | number | string | Uint8Array)
Uint256.toHex(value)
Uint256.toBigInt(value)
Uint256.toNumber(value)  // throws if > MAX_SAFE_INTEGER
Uint256.add(a, b)
Uint256.sub(a, b)
Uint256.mul(a, b)
Uint256.div(a, b)
Uint256.mod(a, b)
Uint256.pow(base, exp)
Uint256.eq(a, b)
Uint256.lt(a, b)
Uint256.gt(a, b)
Uint256.lte(a, b)
Uint256.gte(a, b)
Uint256.isZero(value)
Uint256.max(a, b)
Uint256.min(a, b)
```

## Address Types

| Type | Description | Size |
|------|-------------|------|
| `Address` | Ethereum address | 20 bytes |
| `PublicKey` | secp256k1 public key | 64 bytes (uncompressed) |
| `PrivateKey` | secp256k1 private key | 32 bytes |

```typescript
import * as Address from '@tevm/voltaire/Address';

Address.from(hex | Uint8Array | PublicKey)
Address.toHex(addr)
Address.toChecksum(addr)
Address.isValid(input)
Address.equals(a, b)
Address.isZero(addr)
Address.fromPublicKey(pubKey)
```

```typescript
import * as PrivateKey from '@tevm/voltaire/PrivateKey';

PrivateKey.from(hex | Uint8Array)
PrivateKey.generate()
PrivateKey.toPublicKey(privKey)
PrivateKey.toAddress(privKey)
```

```typescript
import * as PublicKey from '@tevm/voltaire/PublicKey';

PublicKey.from(hex | Uint8Array)
PublicKey.toAddress(pubKey)
PublicKey.compress(pubKey)
PublicKey.decompress(compressed)
```

## Hash Types

| Type | Description | Size |
|------|-------------|------|
| `Hash` | Generic hash | Variable |
| `Bytes32` | 32-byte value | 32 bytes |
| `Keccak256Hash` | Keccak-256 output | 32 bytes |
| `Sha256Hash` | SHA-256 output | 32 bytes |

```typescript
import * as Bytes32 from '@tevm/voltaire/Bytes32';

Bytes32.from(hex | Uint8Array)
Bytes32.toHex(value)
Bytes32.isZero(value)
Bytes32.equals(a, b)
```

## Hex & Bytes

| Type | Description |
|------|-------------|
| `Hex` | 0x-prefixed hex string as bytes |
| `Bytes` | Raw byte array |

```typescript
import * as Hex from '@tevm/voltaire/Hex';

Hex.from(string | Uint8Array | number | bigint)
Hex.toBytes(hex)
Hex.fromBytes(bytes)
Hex.toString(hex)
Hex.concat(...hexes)
Hex.slice(hex, start, end)
Hex.padLeft(hex, length)
Hex.padRight(hex, length)
Hex.isValid(input)
Hex.size(hex)
```

## Transaction Types

| Type | Description | EIP |
|------|-------------|-----|
| `LegacyTransaction` | Pre-EIP-2718 | Legacy |
| `EIP2930Transaction` | Access list | EIP-2930 |
| `EIP1559Transaction` | Fee market | EIP-1559 |
| `EIP4844Transaction` | Blob transaction | EIP-4844 |
| `EIP7702Transaction` | Set EOA code | EIP-7702 |
| `Transaction` | Union of all types | - |

```typescript
import * as Transaction from '@tevm/voltaire/Transaction';

Transaction.from(txData)
Transaction.serialize(tx)
Transaction.sign(tx, privateKey)
Transaction.hash(tx)
Transaction.getSender(signedTx)
Transaction.getType(tx)
Transaction.toRlp(tx)
Transaction.fromRlp(rlpData)
```

## Access Control

| Type | Description | EIP |
|------|-------------|-----|
| `AccessList` | Storage access list | EIP-2930 |
| `AccessListItem` | Single access entry | EIP-2930 |
| `Authorization` | EOA code delegation | EIP-7702 |

```typescript
import * as AccessList from '@tevm/voltaire/AccessList';

AccessList.from(items)
AccessList.toRlp(list)
AccessList.fromRlp(data)
AccessList.merge(list1, list2)
```

```typescript
import * as Authorization from '@tevm/voltaire/Authorization';

Authorization.from({ chainId, address, nonce })
Authorization.sign(auth, privateKey)
Authorization.recover(signedAuth)
Authorization.toRlp(auth)
```

## Encoding

| Type | Description |
|------|-------------|
| `Abi` | ABI encoding/decoding |
| `Rlp` | RLP serialization |
| `Base64` | Base64 encoding |

```typescript
import * as Abi from '@tevm/voltaire/Abi';

Abi.encodeCall({ abi, functionName, args })
Abi.decodeResult({ abi, functionName, data })
Abi.encodeParameters(types, values)
Abi.decodeParameters(types, data)
Abi.encodeEventTopics({ abi, eventName, args })
Abi.decodeEventLog({ abi, data, topics })
Abi.encodePacked(types, values)
Abi.getFunctionSelector(signature)
Abi.getEventSelector(signature)
```

```typescript
import * as Rlp from '@tevm/voltaire/Rlp';

Rlp.encode(value)
Rlp.decode(data)
Rlp.encodeList(items)
Rlp.decodeList(data)
```

## Block Types

| Type | Description |
|------|-------------|
| `Block` | Full block |
| `BlockHeader` | Block header only |
| `BlockHash` | 32-byte block hash |
| `BlockNumber` | Block number (uint64) |

```typescript
import * as Block from '@tevm/voltaire/Block';

Block.from(blockData)
Block.hash(block)
Block.headerHash(header)
Block.toRlp(block)
Block.fromRlp(data)
```

## Receipt & Log Types

| Type | Description |
|------|-------------|
| `Receipt` | Transaction receipt |
| `EventLog` | Event log entry |
| `BloomFilter` | Logs bloom filter |

```typescript
import * as EventLog from '@tevm/voltaire/EventLog';

EventLog.from(logData)
EventLog.decode({ abi, log })
EventLog.matchesTopic(log, topic)
```

## Signature Types

| Type | Description | Size |
|------|-------------|------|
| `Signature` | ECDSA signature | 65 bytes (r,s,v) |
| `CompactSignature` | EIP-2098 compact | 64 bytes |

```typescript
import * as Signature from '@tevm/voltaire/Signature';

Signature.from(hex | { r, s, v })
Signature.sign(messageHash, privateKey)
Signature.recover(messageHash, signature)
Signature.toHex(sig)
Signature.toCompact(sig)
Signature.fromCompact(compactSig)
Signature.verify(messageHash, signature, publicKey)
```

## Denomination Types

| Type | Description | Base Unit |
|------|-------------|-----------|
| `Wei` | Base unit | 1 |
| `Gwei` | Gas unit | 10^9 wei |
| `Ether` | Display unit | 10^18 wei |

```typescript
import * as Wei from '@tevm/voltaire/Wei';
import * as Gwei from '@tevm/voltaire/Gwei';
import * as Ether from '@tevm/voltaire/Ether';

Wei.from(bigint | string)
Wei.toGwei(wei)
Wei.toEther(wei)
Gwei.toWei(gwei)
Ether.toWei(ether)
Ether.format(wei, decimals)
```

## Domain Types (EIP-712)

| Type | Description |
|------|-------------|
| `Domain` | EIP-712 domain |
| `DomainSeparator` | Domain separator hash |
| `TypedData` | Structured data |

```typescript
import * as TypedData from '@tevm/voltaire/TypedData';

TypedData.hash(domain, types, message)
TypedData.sign(domain, types, message, privateKey)
TypedData.verify(domain, types, message, signature, address)
```

## SIWE (Sign-In with Ethereum)

```typescript
import * as Siwe from '@tevm/voltaire/Siwe';

Siwe.createMessage(params)
Siwe.parseMessage(message)
Siwe.verify(message, signature)
```

## Crypto Primitives

| Module | Description |
|--------|-------------|
| `Keccak256` | Keccak-256 hash |
| `Sha256` | SHA-256 hash |
| `Sha3` | SHA-3 hash |
| `Ripemd160` | RIPEMD-160 hash |
| `Blake2` | Blake2b/Blake2s |
| `Secp256k1` | ECDSA operations |
| `Ed25519` | EdDSA operations |
| `Bls12381` | BLS signatures |
| `Bn254` | BN254 curve (alt_bn128) |
| `Kzg` | KZG commitments |
| `Pbkdf2` | Key derivation |
| `Scrypt` | Memory-hard KDF |
| `AesGcm` | AES-GCM encryption |
| `ChaCha20Poly1305` | ChaCha encryption |
| `HdWallet` | HD key derivation |
| `Bip39` | Mnemonic phrases |

```typescript
import * as Keccak256 from '@tevm/voltaire/Keccak256';

Keccak256.hash(data)
Keccak256.hashText(text)
```

```typescript
import * as Secp256k1 from '@tevm/voltaire/Secp256k1';

Secp256k1.sign(messageHash, privateKey)
Secp256k1.verify(messageHash, signature, publicKey)
Secp256k1.recover(messageHash, signature, recoveryId)
Secp256k1.getPublicKey(privateKey)
```
