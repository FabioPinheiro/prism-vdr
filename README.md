# PRISM VDR Indexer

## Structure

Data inside the folder:
- `cardano-21325` 
  - all the Transactions with metadata label 21325.
  - this is all the raw relevant data.
  - index by order of transactions.
  ```json
  {
    "index":0,
    "tx":"<transaction id>",
    "cbor":"<metadata from transaction in CBOR>"
  }
  ```
- `events`
  - all the PRISM Events related with a SSI (Identity).
  - index by the identity.
  ```json
  {
    "tx":"<transaction id>",
    "b":2851<PRISM BLOCK order>,,
    "o":0<Event/operation order inside PRISM BLOCK>,
    "signedWith":"master0",
    "signature":"<hex>",
    "operation":{
      "CreateDidOP":{
        "publicKeys":[
          {
            "CompressedECKey":{
              "id":"master0",
              "usage":"MasterKeyUsage",
              "curve":"secp256k1",
              "data":"<hex>"
            }
          }
        ],
        "services":[],
        "context":[]
      }
    },
    "protobuf":"<operation protobuf data in hex>"
  }
  ```
- `diddoc`
  - the current/latests state of the `did:prism` (based on the SSI)
  - purpose of powering the DID universal resolver for `did:prism`.
  - index by the identity.

- `events`
  - this data is just for **debug purpose**.
  - represent all PRISM Events from the Raw PRISM Blocks (`cardano-21325`) related with a SSI or VDR reference.
  - index by order of PRISM Events.
- `ssi`
  - the current/latests state of the SSI.
  - represent all PRISM Events relative to an SSI reference (`events`).
  - index by the identity.
- `vdr`
  - the current/latests state of the VDR entry.
  - represent all PRISM Events relative to an VDR reference (`events`).
  - index by the identity.

## Run Local 

```shell
docker run --rm -it --entrypoint "sh" --memory="300m" --cpus="1.0" \
  --volume ./mainnet:/data fmgp/prism-indexer \
  -c "java -XX:+UseContainerSupport -jar cardano-prism.jar indexer --token $APIKEY ./data"
```

**Folder structure utils:**

```shell
#rm -rf cardano-21325
rm -rf diddoc
rm -rf events
rm -rf ssi
rm -rf vdr 

mkdir cardano-21325
mkdir diddoc
mkdir events
mkdir ssi
mkdir vdr
```

## TODO

- Blockchain rollback dectetion.
- There is no rollback Mechanism (support to support rollback).
- Using docker to run the Indexer may be overkill.
- Missing index map Event Hash other the Previous relative Event Hash.
  - Implement a constant memory `O(1)` logic to process SSI. (right now is O(number of events))
- Remove the support for cbor metadata and that encodes PRISM operation using text (hex).
- The folder `diddoc` have the DID Document but maybe would be more useful to replace with the did-resolution-result https://w3c.github.io/did-resolution/#did-resolution-result
  - the field `didResolutionMetadata` ... will be contraceptive that can be empty
  - the field `didDocumentMetadata` can have many useful fields: https://w3c.github.io/did-resolution/#did-document-metadata
    - created, updated, deactivated
- Github action:
  -Fix unverified commit. Need GPT
- Questions:
  - should we write the Hash of the PRISM Events in the ops files? (**debug purpose**)
  - should we fetch relevant transactions and store info about it.
  - support for transactions with multiple labels (assuming this is possible)

## Notes

### mainnet

`mainnet` have :
- 6455 Transactions
- 9831 PRISM events/operations
- 312 SSI entries

### preprod

Running from scratch it took `Total time: 375 s (06:15), completed Feb 11, 2025, 8:55:59 PM`
Most of the time was FileSystem IO and also a bit of Network IO.

`preprod` have :
- 16938 Transactions
- 275569 PRISM events/operations
- 71408 SSI entries


---

### FileSystem storage
Using the normal FileSystem as storage is also super inefficient.
From 8.3 MB of uncompressed text to 1.0G in used space... This could also be optimized with special file types.
Hopefully git is very efficient. So this use case is fine. Although there are Git commands that we should avoid because they take time.
But the GitHub interface would probably complain about the number of files. We could have a hashing table structure like using folders for this!

The idea of using GitHub Actions may be more complicated because of this.
Probably we need to do some optimizations in terms of FileSystem or craft the cache of the action very carefully.


We also have the problem of large files
I don't want to use Git LFS (is not a good use case). Since those files are not assets they are mutable datasets!
So the solution will be to have logic to split file in code.

### Github Action

- Cache - https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy
- Cache - https://graphite.dev/guides/github-actions-caching
- Docker - https://aschmelyun.com/blog/using-docker-run-inside-of-github-actions/

