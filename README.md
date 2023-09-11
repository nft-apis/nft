# NFT API

In this tutorial, we will demonstrate how to retrieve data for an NFT collection from [Ethereum]([url](https://ethereum.org/)), Binance Smart Chain (BSC), or any blockchain compatible with the Ethereum Virtual Machine (EVM).

- [ ] Getting started
- [ ] emoji
- [ ] conclusion - with bitquery nft apis and other products
- [ ] Link to our docs
- [ ] Support and other forum
- [ ] CTA - Ask on telegram

## Table of Contents
  - [NFT Ownership API](#nft-ownership-api)
    - [Owners of an NFT 5](#owners-of-an-nft5)
    - [Top Holders of an NFT 3](#top-holders-of-an-nft3)
  - [NFT Transfer API](#nft-transfer-api)
    - [Daily NFT Transfers](#daily-nft-transfers)
    - [Latest Transfers of an NFT](#latest-transfers-of-an-nft)
    - [Latest NFT transfers from and to a User](#latest-nft-transfers-from-and-to-a-user)
    - [Subscribing to Realtime NFT Transfers](#subscribing-to-realtime-nft-transfers)
  - [NFT Calls API](#nft-calls-api)
    - [Latest Calls for an NFT](#latest-calls-for-an-nft)
  - [NFT Trades API](#nft-trades-api)
    - [Latest NFT Trades on Opensea](#latest-nft-trades-on-opensea)
    - [Top Traded NFTs on Opensea](#top-traded-nfts-on-opensea)
    - [Total Buy and sell of specific NFT on Opensea](#total-buy-and-sell-of-specific-nft-on-opensea)
    - [**Latest NFT buyer on Opensea**](#latest-nft-buyer-on-opensea)
    - [Specific Buyer stats of an NFT on Opensea](#specific-buyer-stats-of-an-nft-on-opensea)
    - [Latest NFT Trades on Ethereum for Seaport protocol](#latest-nft-trades-on-ethereum-for-seaport-protocol)
    - [Get Latest Trades of an Address](#get-latest-trades-of-an-address)
    - [Get Top Traded NFT Tokens](#get-top-traded-nft-tokens)
  - [NFT Metadata API](#nft-metadata-api)
    - [Get the creator of an NFT](#get-the-creator-of-an-nft)
    - [**Get the Metadata of an NFT**](#get-the-metadata-of-an-nft)
  - [NFT Collection API](#nft-collection-api)
    - [Get All NFTs in a collection](#get-all-nfts-in-a-collection)
    - [**Get Latest NFT Listing**](#get-latest-nft-listing)
    - [Get All Token Holders of a Collection](#get-all-token-holders-of-a-collection)

## NFT Ownership API

### [Owners of an NFT 5](https://ide.bitquery.io/Who-owns-specific-NFT)

To get ownership of an NFT token, we need to provide its token address and ID of the specific NFT. It will provide all the owners if we don’t provide `limit` and `orderBy`. By providing this field, we are getting the latest owner of the NFT.

This API uses the `BalanceUpdates` method, which tracks all balance updates. Additionally, you can get similar results using the [Transfers API](https://ide.bitquery.io/Onwership-of-an-nft-token-using-transfers-api_1).

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    BalanceUpdates(
      where: {BalanceUpdate: {Id: {eq: "9990"}}, Currency: {SmartContract: {is: "0x8a90cab2b38dba80c64b7734e58ee1db38b8992e"}}}
      limit: {count: 10}
      orderBy: {descendingByField: "Balance"}
    ) {
      BalanceUpdate {
        Address
      }
      Balance: sum(of:BalanceUpdate_Amount, selectWhere:{gt:"0"})
    }
  }
}
```

### [Top Holders of an NFT 3](https://ide.bitquery.io/top-token-holders-of-Moonwalker-NFT)

To get the top holders of an NFT we provide the NFT address in the Currency `SmartContract` field. We limit the list to 10 owners ordered by the `balance`.

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    BalanceUpdates(
      orderBy: {descendingByField: "Balance"}
      limit: {count: 10}
      where: {Currency: {SmartContract: {is: "0x7dD4F223D9155F412790D696Fa30923489d4Ad34"}}}
    ) {
      BalanceUpdate {
        Address
      }
      Balance: sum(of: BalanceUpdate_Amount, selectWhere: {gt: "0"})
    }
  }
}
```

## NFT Transfer API

### [Daily NFT Transfers](https://ide.bitquery.io/NFT-Token-Transfers-By-Date)

The following query provides the daily NFT transfers using the Transfers API.

```graphql
{
  EVM(dataset: combined network: eth){
    Transfers(
      orderBy: {descendingByField: "count"}
      limit: {offset: 10 count: 0}
      where: {
        Block: {Date: {since: "2023-05-02" till: "2023-05-09" }}
        Transfer: {Currency: {Fungible: false}}}
    ){
      Transfer {
        Currency {
          Symbol
          SmartContract
        }
      }
      count
      senders: uniq(of: Transfer_Sender method: approximate)
      receivers: uniq(of: Transfer_Receiver method: approximate)
      ids: uniq(of: Transfer_Id method: approximate)
    }
  }
}
```

### [Latest Transfers of an NFT](https://ide.bitquery.io/latest-nft-transfers)

To get the latest transfers, we will provide the NFT token contract address; if you want to track the transfer of a specific NFT, then you can also provide the `ID` of the NFT with the token address.

```graphql
{
  EVM(dataset: archive, network: eth) {
    Transfers(
      where: {Transfer: {Currency: {SmartContract: {is: "0xdba45c28b32f2750bdc3c25d6a0118d8e1c8ca80"}}}}
      limit: {count: 10}
      orderBy: {descending: Block_Time}
    ) {
      Transfer {
        Amount
        Currency {
          Name
          Symbol
        }
        Receiver
        Sender
        Type
        Id
        URI
        Data
      }
    }
  }
}
```

### [Latest NFT transfers from and to a User](https://ide.bitquery.io/latest-nft-transfers-by-a-user)

To get the latest NFT transfers of a user, you can use Transfers APIs; in the following API, we are getting all transfers where the user is the sender and where the user is the receiver.

You can also get transfers of more than 1 user by using `in: []` operators instead of `is: {}`

```graphql
{
  EVM(dataset: archive, network: eth) {
    sent: Transfers(
      where: {Transfer: {Sender: {is: "0x415bdfed5a7c490e1a89332648d8eb339d4eea69"}}}
      limit: {count: 10}
      orderBy: {descending: Block_Time}
    ) {
      Transfer {
        Amount
        Currency {
          Name
          Symbol
        }
        Receiver
        Sender
        Type
        Id
        URI
      }
    }
    recieved: Transfers(
      where: {Transfer: {Receiver: {is: "0x415bdfed5a7c490e1a89332648d8eb339d4eea69"}}}
      limit: {count: 10}
      orderBy: {descending: Block_Time}
    ) {
      Transfer {
        Amount
        Currency {
          Name
          Symbol
        }
        Receiver
        Sender
        Type
        Id
        URI
      }
    }
  }
}
```

### [Subscribing to Realtime NFT Transfers](https://ide.bitquery.io/Subscription-WebSocket---Latest-NFT-Transfers)

Using our Streaming APIs, you can also subscribe to changes on blockchains. We use a GrpahQL subscription, which is similar to WebSockets. For example, in the following API, we are subscribing to the latest transfers of [0xdba45c28b32f2750bdc3c25d6a0118d8e1c8ca80](https://explorer.bitquery.io/ethereum/token/0xdba45c28b32f2750bdc3c25d6a0118d8e1c8ca80) NFT token.

```graphql
subscription {
  EVM(network: eth, trigger_on: head) {
    Transfers(
      where: {Transfer: {Currency: {SmartContract: {is: "0xdba45c28b32f2750bdc3c25d6a0118d8e1c8ca80"}}}}
      limit: {count: 10}
      orderBy: {descending: Block_Time}
    ) {
      Transfer {
        Amount
        Currency {
          Name
          Symbol
        }
        Receiver
        Sender
        Type
        Id
        URI
        Data
      }
    }
  }
}
```

## NFT Calls API

### [Latest Calls for an NFT](https://ide.bitquery.io/Smart-contract-calls-to-an-nft-contract)

To check the latest Smart contract calls made on NFT token contracts, you can use our `smartContractCalls` API.

```graphql
{
  EVM {
    Calls(
      limit: {count: 10}
      orderBy: {descending: Block_Time}
      where: {Call: {To: {is: "0x60e4d786628fea6478f785a6d7e704777c86a7c6"}}}
    ) {
      Call {
        From
        Gas
        GasUsed
        To
        Value
      }
      Transaction {
        Hash
      }
      Arguments {
        Name
        Value {
          ... on EVM_ABI_Integer_Value_Arg {
            integer
          }
          ... on EVM_ABI_String_Value_Arg {
            string
          }
          ... on EVM_ABI_Address_Value_Arg {
            address
          }
          ... on EVM_ABI_BigInt_Value_Arg {
            bigInteger
          }
          ... on EVM_ABI_Bytes_Value_Arg {
            hex
          }
          ... on EVM_ABI_Boolean_Value_Arg {
            bool
          }
        }
      }
    }
  }
}
```

## NFT Trades API

### [Latest NFT Trades on Opensea](https://ide.bitquery.io/Latest-OpenSea-Trades)

In the following query, we are getting the latest Opensea trades by tracking the [Seaport protocol](https://opensea.io/blog/articles/introducing-seaport-protocol) (Here seaport_v1.4 means all versions of seaport) and all transactions sent to Opensea’s seaport contract `0x00000000000000adc04c56bf30ac9d3c0aaf14dc`.

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {Trade: {Dex: {ProtocolName: {in: "seaport_v1.4"}}}, Transaction: {To: {is: "0x00000000000000adc04c56bf30ac9d3c0aaf14dc"}}}
      limit: {count: 10}
    ) {
      Trade {
        Buy {
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
          Amount
          Buyer
          Ids
          Price
          URIs
        }
        Sell {
          Currency {
            Name
            ProtocolName
            Symbol
            Decimals
            Fungible
            SmartContract
          }
          Amount
          Buyer
          Ids
          URIs
        }
      }
    }
  }
}
```

### [Top Traded NFTs on Opensea](https://ide.bitquery.io/Top-Traded-NFTs-on-Opensea)

We can aggregate trading vol, trade count, buyer, seller, and nfts and sort them based on trade count in the following query to get the most traded NFT on OpenSea.

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {Trade: {Dex: {ProtocolName: {in: "seaport_v1.4"}}}, Transaction: {To: {is: "0x00000000000000adc04c56bf30ac9d3c0aaf14dc"}}}
      orderBy: {descendingByField: "count"}
      limit: {count: 10}
    ) {
      tradeVol: sum(of: Trade_Buy_Amount)
      count
      buyers: count(distinct: Trade_Buy_Buyer)
      seller: count(distinct: Trade_Buy_Seller)
      nfts: count(distinct: Trade_Buy_Ids)
      Trade {
        Buy {
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
        }
      }
    }
  }
}
```

### [Total Buy and sell of specific NFT on Opensea](https://ide.bitquery.io/Total-buy-sell-of-an-NFT-on-opensea)

To get the stats for specific NFT we need to add the currency contract address in the above query. For example, check the query below.

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {Trade: {Dex: {ProtocolName: {in: "seaport_v1.4"}}, Buy: {Currency: {Fungible: false}}}, Transaction: {To: {is: "0x00000000000000adc04c56bf30ac9d3c0aaf14dc"}}}
      orderBy: {descendingByField: "count"}
      limit: {count: 10}
    ) {
      tradeVol: sum(of: Trade_Buy_Amount)
      count
      buyer: count(distinct: Trade_Buy_Buyer)
      seller: count(distinct: Trade_Buy_Seller)
      nfts: count(distinct: Trade_Buy_Ids)
      Trade {
        Buy {
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
        }
      }
    }
  }
}
```

### **[Latest NFT buyer on Opensea](https://ide.bitquery.io/Top-buyers-of-NFTs-on-Opensea)**

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {Trade: {Dex: {ProtocolName: {in: "seaport_v1.4"}}, Buy: {Currency: {Fungible: false}}}, Transaction: {To: {is: "0x00000000000000adc04c56bf30ac9d3c0aaf14dc"}}}
      orderBy: {descendingByField: "count"}
      limit: {count: 10}
    ) {
      count
      uniq_tx: count(distinct: Transaction_Hash)
      Block {
        first_date: Time(minimum: Block_Date)
        last_date: Time(maximum: Block_Date)
      }
      nfts: count(distinct: Trade_Buy_Ids)
      difffernt_nfts: count(distinct: Trade_Buy_Currency_SmartContract)
      total_money_paid: sum(of: Trade_Sell_Amount)
      Trade {
        Buy {
          Buyer
        }
      }
    }
  }
}
```

### [Specific Buyer stats of an NFT on Opensea](https://ide.bitquery.io/Top-buyer-of-specific-NFT)

In this query, we will get specific buyer stats for a specific NFT on Opensea.

```graphql
query MyQuery {
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      where: {Trade: {Dex: {ProtocolName: {in: "seaport_v1.4"}}, Buy: {Currency: {SmartContract: {is: "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d"}}, Buyer: {is: "0x2f9ecaa66e12b6168996a6b80cda9bb142f80dd0"}}}, Transaction: {To: {is: "0x00000000000000adc04c56bf30ac9d3c0aaf14dc"}}}
      orderBy: {descendingByField: "count"}
      limit: {count: 10}
    ) {
      count
      uniq_tx: count(distinct: Transaction_Hash)
      Block {
        first_date: Time(minimum: Block_Date)
        last_date: Time(maximum: Block_Date)
      }
      nfts: count(distinct: Trade_Buy_Ids)
      Trade {
        Buy {
          Buyer
          Currency {
            Name
            ProtocolName
            Symbol
            Fungible
            SmartContract
          }
        }
      }
    }
  }
}
```

### [Latest NFT Trades on Ethereum for Seaport protocol](https://ide.bitquery.io/latest-NFT-trades-on-Ethereum-network)

In the following query, we are getting all NFT trades for `Seaport v1.4` protocol. Many marketplaces utilize the Seaport protocol; we can add a Smart contract in Trade → Dex → SmartContract to get a specific marketplace for this protocol.

```graphql
query MyQuery {
  EVM {
    DEXTrades(
      limit: {offset: 0, count: 10}
      orderBy: {descendingByField: "Block_Time"}
      where: {Trade: {Dex: {ProtocolName: {is: "seaport_v1.4"}}}}
    ) {
      Trade {
        Dex {
          ProtocolName
        }
        Buy {
          Price
          Seller
          Buyer
          Currency {
            HasURI
            Name
            Fungible
            SmartContract
          }
        }
        Sell {
          Price
          Amount
          Currency {
            Name
          }
          Buyer
          Seller
        }
      }
      Transaction {
        Hash
      }
      Block {
        Time
      }
    }
  }
}
```

### [Get Latest Trades of an Address](https://ide.bitquery.io/NFT-trades-of-an-address)

In the following query, we are getting the latest NFT trades of an address.

```graphql
query MyQuery {
  EVM(dataset: combined) {
    DEXTrades(
      limit: {offset: 0, count: 10}
      orderBy: {descendingByField: "Block_Time"}
      where: {Trade: {Buy: {Buyer: {is: "0x6afdf83501af209d2455e49ed9179c209852a701"}, Currency: {Fungible: false}}}}
    ) {
      Trade {
        Dex {
          ProtocolName
          OwnerAddress
          Delegated
          DelegatedTo
          ProtocolName
          SmartContract
        }
        Buy {
          Price
          Seller
          Buyer
          Currency {
            Symbol
            HasURI
            Name
            Fungible
            SmartContract
          }
          Ids
          OrderId
          URIs
        }
        Sell {
          Price
          Amount
          Currency {
            Name
          }
          Buyer
          Seller
        }
      }
      Transaction {
        Hash
      }
      Block {
        Time
      }
    }
  }
}
```

### [Get Top Traded NFT Tokens](https://ide.bitquery.io/Top-traded-NFT-tokens-in-a-month)

Let’s get the most traded NFTs of the month using the following query.

```graphql
{
  EVM(dataset: combined, network: eth) {
    DEXTrades(
      orderBy: {descendingByField: "count"}
      limit: {offset: 0, count: 10}
      where: {Block: {Date: {since: "2023-05-01", till: "2023-05-28"}}, Trade: {Buy: {Currency: {Fungible: false}}, Sell: {Currency: {Fungible: true}}}}
    ) {
      Trade {
        Buy {
          Currency {
            Symbol
            SmartContract
          }
          min_price: Price(minimum: Trade_Buy_Price)
          max_rice: Price(maximum: Trade_Buy_Price)
        }
        Sell {
          Currency {
            Symbol
            SmartContract
          }
        }
      }
      buy_amount: sum(of: Trade_Buy_Amount)
      sell_amount: sum(of: Trade_Sell_Amount)
      count
    }
  }
}
```

## NFT Metadata API

### [Get the creator of an NFT](https://ide.bitquery.io/Owner-of-a-contract)

We will use the current dataset (with endpoint - `https://graphql.bitquery.io/`) to get the address that created the transaction which created the contract `0x6339e5E072086621540D0362C4e3Cea0d643E114`

```graphql
{
  ethereum {
    smartContractCalls(
      options: {desc: "block.height", limit: 100, offset: 0}
      smartContractMethod: {is: "Contract Creation"}
      smartContractAddress: {is: "0x6339e5E072086621540D0362C4e3Cea0d643E114"}
    ) {
      block {
        height
        timestamp {
          time
        }
      }
      creator: caller {
        address
        annotation
      }
      transaction {
        hash
      }
      smartContract {
        contractType
        address {
          address
          annotation
        }
        currency {
          name
          symbol
          decimals
          tokenType
        }
      }
    }
  }
}
```

### **[Get the Metadata of an NFT](https://ide.bitquery.io/NFT-metadata_1_1)**

```graphql
{
  EVM(dataset: archive, network: eth) {
    Transfers(
      where: {Transfer: {Currency: {SmartContract: {is: "0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D"}}, Id: {eq: "4226"}}}
      limit: {count: 1, offset: 0}
      orderBy: {descending: Block_Number}
    ) {
      Transfer {
        Currency {
          SmartContract
          Name
          Decimals
          Fungible
          HasURI
          Symbol
        }
        Id
        URI
        Data
        owner: Receiver
      }
    }
  }
}
```

## NFT Collection API

### [Get All NFTs in a collection](https://ide.bitquery.io/Get-all-NFTs-for-a-collection)

```graphql
{
  EVM(dataset: archive, network: eth) {
    Transfers(
      where: {Transfer: {Currency: {SmartContract: {is: "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d"}}}}
      limitBy: {by: Transfer_Id, count: 1}
      limit: {count: 1000, offset: 0}
      orderBy: {descending: Transfer_Id}
    ) {
      Transfer {
        Currency {
          SmartContract
          Name
        }
        Id
        URI
        Data
      }
    }
  }
}
```

### **[Get Latest NFT Listing](https://ide.bitquery.io/Latest-Listing-on-Opensea)**

```graphql
{
  ethereum {
    arguments(
      smartContractEvent: {is: "SaleCreatedEvent"}
      smartContractAddress: {is: "0xa2707b069EEbca7f8Ae133162f19FE720D3aAA58"}
      options: {limit: 100}
    ) {
      block {
        height
      }
      acceptFiat: any(of: argument_value, argument: {is: "acceptFiat"})
      priceInWei: any(of: argument_value, argument: {is: "priceInWei"})
      tokenId: any(of: argument_value, argument: {is: "tokenId"})
      acceptFiat: any(of: argument_value, argument: {is: "acceptFiat"})
      tokenContractAddress: any(
        of: argument_value
        argument: {is: "tokenContractAddress"}
      )
      transaction {
        hash
      }
    }
  }
}
```

### [Get All Token Holders of a Collection](https://ide.bitquery.io/Fixed---All-token-holders-of-ERC-1165-collection)

To get the token holders of a collection, we use the `BalanceUpdates` method to query all the addresses that had added the NFT whose `SmartContract` is mentioned in the query. To get more wallets, increase the `count` .

```graphql
{
  EVM(dataset: archive, network: eth) {
    BalanceUpdates(
      where: {Currency: {SmartContract: {is: "0x23581767a106ae21c074b2276d25e5c3e136a68b"}}}
      limitBy: {by: BalanceUpdate_Address, count: 1}
      limit: {count: 1000}
      orderBy: {descendingByField: "sum"}
    ) {
      sum(of: BalanceUpdate_Amount, selectWhere: {gt: "0"})
      BalanceUpdate {
        Address
      }
    }
  }
}
```
