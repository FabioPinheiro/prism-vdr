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

## Run Local with Coursier

```shell
  cs launch app.fmgp::cardano-prism-cli:0.1.0-M44 -M fmgp.did.method.prism.cli.PrismCli -- \
    indexer in-memory --token preprod9EGSSMf6oWb81qoi8eW65iWaQuHJ1HwB  ./preprod
```

## Run Local DOCKER (OLD)

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
- Remove the support for cbor metadata and that encodes PRISM operation using text (hex).
- The folder `diddoc` have the DID Document but maybe would be more useful to replace with the did-resolution-result https://w3c.github.io/did-resolution/#did-resolution-result
  - the field `didResolutionMetadata` ... will be contraceptive that can be empty
  - the field `didDocumentMetadata` can have many useful fields: https://w3c.github.io/did-resolution/#did-document-metadata
    - created, updated, deactivated
- Github action:
  -Fix unverified commit. Need GPT

## Notes


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

