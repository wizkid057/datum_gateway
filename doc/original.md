# Original, with a Bitcoin receipt

Anyone can copy a README and replace the words "official repository." DATUM's upstream declaration comes with something more interesting: a SHA-256 commitment published in a Bitcoin transaction.

The exact bytes of [original.txt](original.txt) identify **[OCEAN-xyz/datum_gateway](https://github.com/OCEAN-xyz/datum_gateway)** as the original and only official DATUM Gateway repository, state that official DATUM supports Bitcoin only, and record Jason Hughes's authorship of DATUM for OCEAN and the Gateway's October 2024 MIT-licensed release.

Jason committed that file's SHA-256 hash in the first output of this Bitcoin transaction:

```text
93c11a0bab50f20edebc6d42455312a3574ccbef34abeff8366947389b39beea
```

The transaction is recorded in **Bitcoin block 970877**, with block hash:

```text
000000000000000000008a0f66c5b82c519836caba0432ca06f41d7be09702d8
```

The declaration now has a permanent reference point in Bitcoin's history.

## Why this is fun—and useful

Git history, repository names, and website text can be rewritten. This transaction's commitment has a fixed position in Bitcoin's shared history. Under SHA-256's security assumptions, the matching file is evidence that this exact declaration existed by that block's inclusion.

Changing the repository URL, author, or even one byte of the declaration changes its hash. A modified declaration needs its own commitment; a later transaction cannot inherit this one's earlier block position simply by copying the documentation. Someone can copy the original file and its receipt, but that copy still points to **OCEAN-xyz/datum_gateway**.

That gives readers a durable, independently verifiable reference beyond whichever README they happen to be viewing. Bitcoin mining software using Bitcoin to preserve its own upstream declaration is a particularly fitting receipt.

## But why is the transaction confirmed two years after release?

DATUM Gateway was released in October 2024. At the time, putting an upstream declaration into the Bitcoin blockchain simply did not seem necessary. The project's authorship, purpose, and public code history were already there for anyone to see.

Since then, multiple forks have made claims contrary to the actual history of the codebase. That made preserving an explicit declaration worthwhile. Committing it to Bitcoin seemed like a very "Bitcoin" way of locking in the truth: publish the statement, preserve its exact bytes, and give everyone a way to verify the record for themselves.

The transaction records this declaration in 2026; the project's release and development history date back earlier. The commitment gives that declaration a lasting reference that a later fork cannot change by editing a README.

## A little Bitcoin history in the input

Jason also chose an input with some mileage on it:

- The transaction spends output **46** of transaction `a062e89cb3c136c2e315f3c118c91f62f82bd2ae446c5ddf19854dca47599454`, confirmed on **September 6, 2014**, in **block 319464**—an output left unspent for about **12 years**.
- That output belonged to address `12LPeeQmrUKbMKcamxLSaRqwBHKoBSwJz3`, whose earliest recorded transaction, `a498f354ac4dc92946bddb87f6f1aea35a1ba6de358b5f72fa21585c99926c99`, was confirmed on **June 13, 2012**, in **block 184410**—about **14 years** earlier.

A twelve-year-old UTXO put back to work to say, "Yes, this is the original." A little extra history behind the declaration.

## Verify it yourself

From the repository root, hash the file without changing it:

```bash
sha256sum doc/original.txt
```

Expected SHA-256:

```text
f472691ca8fcf3c4f304f381ea1ae960feed85e711c1651a404f0f58e6ec7aa1
```

Inspect the transaction's **first output (`vout[0]`)** using your own Bitcoin node. Its script is `OP_RETURN`, followed by a push of those 32 hash bytes. The complete script in hexadecimal is:

```text
6a20f472691ca8fcf3c4f304f381ea1ae960feed85e711c1651a404f0f58e6ec7aa1
```

Here, `6a` is `OP_RETURN`, and `20` specifies a 32-byte data push. The transaction contains the hash, not the document itself. Verify that the hash matches and that the transaction is included in block 970877 using your own fully validating Bitcoin node.

**Do not modify `original.txt`.** Whitespace, line endings, encoding, punctuation, and the final newline are all part of the committed bytes. Keep the original file intact; explanations and later updates belong in this Markdown document or a separate declaration with its own commitment.

## What the receipt establishes

The commitment timestamps the exact declaration in Bitcoin block 970877. It does not independently prove every statement in it, establish legal ownership, certify a software download, or prevent anyone from making a fork. A block's timestamp is also not an exact document creation time; the useful evidence is inclusion and ordering in the Bitcoin chain.

The MIT license and the freedom to fork remain unchanged. This adds a checkable historical reference for the original upstream—and makes a copied "we were first" claim a little harder to wave around without evidence.
