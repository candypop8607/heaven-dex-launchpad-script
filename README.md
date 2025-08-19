## 🌌 Heaven DEX Launchpad – Solana Reference

This repository contains reference code for working with Solana Versioned Transactions on the Heaven DEX Launchpad.

It demonstrates how to:

- Fetch and resolve Address Lookup Tables (LUTs of heaven DEX).

- Decompile a Solana versioned transaction.

- Rebuild the transaction from a TransactionMessage using LUTs.

This repo is intended as a developer reference for anyone building tools or integrations with Heaven DEX or other Solana-based projects.

Main code is kept as private in following example.

## Example flow

```
const connection = new Connection("https://mainnet.helius-rpc.com/?api-key=???", "confirmed")
  const wallet = new PublicKey("7mFgFYwqGBNJcKRYdjeH9WoLKiAsjxnM6D7eKV1CnRfb")
  const { mint, tx: serializedTx } = await makeCreationTx()
  const txBuffer = Buffer.from(serializedTx, "base64");
  const versionedTx = VersionedTransaction.deserialize(txBuffer);

  const lookupTableAccounts: AddressLookupTableAccount[] = await Promise.all(
    versionedTx.message.addressTableLookups.map(async (lookup) => {
      const accountInfo = await connection.getAccountInfo(lookup.accountKey);
      return new AddressLookupTableAccount({
        key: lookup.accountKey,
        state: AddressLookupTableAccount.deserialize(accountInfo!.data),
      });
    })
  );

  const decompiledMessage = TransactionMessage.decompile(
    versionedTx.message,
    { addressLookupTableAccounts: lookupTableAccounts }
  );

  const { blockhash } = await connection.getLatestBlockhash();
  const compiledV0 = new TransactionMessage({
    payerKey: decompiledMessage.payerKey,        // keep original fee payer
    recentBlockhash: blockhash,           // refresh blockhash
    instructions: decompiledMessage.instructions // resolved TransactionInstruction[]
  }).compileToV0Message(lookupTableAccounts);

  const vTx = new VersionedTransaction(compiledV0);

```

### 👤 Author
#### Twitter: [@Rabnail_SOL](https://twitter.com/Rabnail_SOL)   
#### Telegram: [@Rabnail_SOL](https://t.me/Rabnail_SOL)   
