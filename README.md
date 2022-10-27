# ComposeDB Workshop @ PL Summit 2022 - Lisbon
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg)](https://lbesson.mit-license.org/)
[![](https://img.shields.io/badge/Chat%20on-Discord-orange.svg?style=flat)](https://discord.gg/6VRZpGP)
[![Twitter](https://img.shields.io/twitter/follow/ceramicnetwork?label=Follow&style=social)](https://twitter.com/ceramicnetwork)
> ATTN: ComposeDB is currently in active development and composites CANNOT be deployed on mainnet!

***What does this workshop cover:***
1. Setup Ceramic node incl. ComposeDB developer preview with relations support
2. Create and deploy your first composites
3. Add and query data
4. ✨Ceramic Magic ✨

Oh and btw... we'd ❤️ to hear your feedback - ping us on Discord or submit directly through our [FORUM](https://forum.ceramic.network/c/feedback-on-ceramic/10)

***So, lets get started!*** 🚀

## 1. Installation

Install the necessities - `@next` ensures you are running the latest developer preview:\
`pnpm install --global @composedb/cli@next @ceramicnetwork/cli@next`

Now to create a new PK & corresponding DID to get our node setup. \
`composedb did:generate-private-key` 

For demo purposes and to follow along with the tutorial you can store the PK in `priv.key`. Just be careful *NOT TO COMMIT* this file by accident. Alternatively, you can also set a new ENV variable `DID_PRIVATE_KEY`. Doing this you can just omit `--did-private-key` from the example commands.

> *ATTN*: ALWAYS keep your private key safe and copy the DID into the node config file located under `~/.ceramic/daemon.config.json` which will prepare the node for future CLI interactions. If you don't have a config file in your directory yet, simply start the node once via `ceramic daemon` and exit again. This will create a default config file:

Node config example:

```json
{
  "anchor": {},
  "http-api": {
    "cors-allowed-origins": [
      ".*"
    ],
    "admin-dids": ["did:key:INSERT_DID_KEY_HERE"]
  },
  "ipfs": {
    "mode": "bundled"
  },
  "logger": {
    "log-level": 2,
    "log-to-files": false
  },
  "metrics": {
    "metrics-exporter-enabled": false,
    "metrics-port": 9090
  },
  "network": {
    "name": "testnet-clay"
  },
  "node": {},
  "state-store": {
    "mode": "fs",
    "local-directory": "/Users/Ceramic/.ceramic/statestore/"
  },
  "indexing": {
    "db": "sqlite:///Users/Ceramic/.ceramic/indexing.sqlite",
    "allow-queries-before-historical-sync": true,
    "models": []
  }
}
```

After adding the DID, lets start the node and on to the build steps: \
`export CERAMIC_ENABLE_EXPERIMENTAL_COMPOSE_DB="true"; ceramic daemon`

## 2. Composite Build Steps - Your First Relation
> Ensure the Cearmic Daemon is running and proper indexing environment variables are set export CERAMIC_ENABLE_EXPERIMENTAL_COMPOSE_DB=true WARNING: if you have previous models indexed that you don’t care about anymore you’ll need to remove the sqlite file if you no longer want Ceramic to index them. The daemon will not start if they exist in the database but are not referenced in the config.

1. Create or edit `AddressBook.graphql` schema
2. Create a composite from the AddressBook schema: \
`composedb composite:create schemas/AddressBook.graphql --output eth-address-book.json --did-private-key $(cat priv.key)`
3. Create or edit `AddressBookEntry.graqhql` schema: \
`composedb composite:create schemas/AddressBookEntry.graphql --output=eth-address-book-entries.json --did-private-key $(cat priv.key)`
4. Merge the composites into a single encoded file: \
`composedb composite:merge eth-address-book.json eth-address-book-entries.json --output=demo-address-book-merged.json`
5. Deploy the merged composite. This will add the models to your local node and start indexing automatically: \
`composedb composite:deploy demo-address-book-merged.json`

***That's it! We are ready to play with data!*** 🙌

## 3. Runtime Steps - Say Hello to GRAPHIQL

If you made it this far, I'm proud of you! Now comes the really fun part:

1. Compile a runtime representation to use (this would be used by our application to interact with the node): \
`composedb composite:compile demo-address-book-merged.json demo-runtime.json`

2. In lieu of a front-end, ComposeDB ships with GraphiQL to get you started:
`composedb graphql:server --graphiql --port=35000 demo-runtime.json --did-private-key=$(cat priv.key)`

You should be able to [CONNECT](http://localhost:35000/graphql) now!

### Query Composite:
```
query{
    addressBookIndex(first:10) {
        edges{
            node{
                id
                addressBookName
        }}
    
    }
}
```
Since we don't have data nothing will be returned, but if you don't get an error you are ready to go!

### Create new address book
```
mutation CreateNewAddressBook($i: CreateAddressBookInput!) {
    createAddressBook(input: $i) {
        document {
            id
            addressBookName
        }
    }
}
```

***Query Variables:***
```
{
    "i": {
        "content": {
            "addressBookName": "PL Summit"
        }
    }
}
```


### Create new address book entry

```
mutation CreateNewAddressBookEntry ($i: CreateAddressBookEntryInput!) {
    createAddressBookEntry(input: $i) {
        document {
            id
            entryName
            walletAddresses {
                address
                blockchainNetwork
            }
        }
    }
}
```

***Query Variables:***
```
{
  "i": {
    "content": {
      "addressBook": "INSERT CREATED ADDRESS BOOK ID HERE",
      "entryName": "my fantastic entry",
      "walletAddresses": {
          "address": "0x00",
          "blockchainNetwork": "ethereum"
      }
    }
  }
}
```

### Query Composite Data:
```
query {
    addressBookEntryIndex(first: 5) {
        edges{
            node {
                book {
                    id
                    addressBookName
                }
                entryName
                walletAddresses {
                    address
                    blockchainNetwork
                }
            }
        }
    }
}
```

And thats all folks. You are now officialy a ComposeDB PRO. Go back and have a look at the schemas and play around with them or try to create your own schemas and models.

## 4. Bonus content

- Start a deep dive with the [COMPOSE DB DOCS](https://composedb.js.org)
- Recap [VIDEO](https://drive.google.com/file/d/1X6KCKLWXj6r8XKZlKic1rpomHJBZGxP6/view?usp=sharing) for later watching
> WARNING: not a 100% match to the above & might be a bit outdated but captures the gist of the demo
- Come and join our [COMMUNITY FORM](https://forum.ceramic.network)
- Node deployment [TEMPLATE (EXPERIMENTAL!!!)](https://github.com/ceramicstudio/ceramic-infra-poc)