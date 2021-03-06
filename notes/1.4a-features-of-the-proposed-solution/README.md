# \[Archive] 1.4a - OLD VERSION

## I'm going to delete this file when I've finished transferring all it's info into [the new 1.4a File](../../analysis/1.4a-features-of-proposed-solution.md), as it's badly structured and I don't like it.

## Transactions

### The Structure of Transactions and Digital Signatures

Transactions on this blockchain will abide by a few rules, rules which form the system that allow for digital signatures to take place and therefore transactions on the whole.

These rules are the following:

* Wallets (and therefore users) are identified by their public keys
* The transactions themselves are signed using a wallet's private key and then their signature and transaction combo can be verified using that wallet's public key. The point of this is to only allow the sender to create transactions where they send an item, but anyone can check and verify these transactions.
* Transactions should include a timestamp of when they were created by the sender, as the block also contains a timestamp, if the transaction is included on a block that is timestamped to outside of a set time frame then the transaction should be counted as null and void.
* Transactions should then also obviously include where the coins/proof of ownership should be sent to, and what is being sent.
* The&#x20;

Following these rules gives us an idea of what each transaction should look like:

| Sender's public key | Transaction                | Recipient's public key | Timestamp                    | Sender Signature  |
| ------------------- | -------------------------- | ---------------------- | ---------------------------- | ----------------- |
| b94d27b9            | {type: coin, quantity: 50} | j45n63m3               | Thu Mar 17 2022 19:06:50 GMT | 821a643d5ebf18ee9 |

In JSON format that would look something like the following:

```
{
    Sender: "b94d27b9",
    Transaction: [{type: coin, quantity: 50}],
    Recipient: "j45n63m3",
    Timestamp: "Thu Mar 17 2022 19:06:50 GMT",
    Sender-Signature: "821a643d5ebf18ee9"
}    
```

It's important to mention that the Sender-Signed hash of the data does not include itself, otherwise that would create a recursion error and would be impossible to destruct. Instead the Sender-Signed hash is a hash of everything except itself in a separate JSON object than the one in the blockchain.

Therefore, in this scenario the sender would be hashing and signing the following:

```
{
    Sender-Key: "b94d27b9",
    Transaction: [{type: coin, quantity: 50}],
    Recipient-Key: "j45n63m3",
    Timestamp: "Thu Mar 17 2022 19:06:50 GMT",
}
```

This is very important to bear in mind when writing the node mining software as if the whole json transaction is validated it will never validate successfully, and instead the section of the transaction referenced above is the part that should be validated according to the Sender-signed hash.

The other alternative to this is to construct the transaction object in two parts: the signature and the data. This would be structured similarly to how a typical communication request is structured, with a header and a body.

```
{
    header: {
        sender: "b94d27b9",
        signature: "821a643d5ebf18ee9",    
    },
    body: {
        recipient: "j45n63m3",
        Transaction:
            [{type: coin, quantity: 50}],
        Timestamp: "Thu Mar 17 2022 19:06:50 GMT",
    }
}
```

The final construction has not yet been decided, but it should abide by the above rules.

## Types of Transactions and Valid Data

### There will initially be four main types of data that can be sent over a transaction:

* Coin - Any type of cryptocurrency.
* Certificate - A certificate of ownership
* Creation - How things are created on the blockchain
* Custom - Any other json data, this is less strict than the other types and therefore less secure.

### Coins

This is what is commonly referred to as a cryptocurrency, and is the data that represents money and allows the whole blockchain to run. There will be one key coin that will be the coin the default node software will be paid in, however if a user was to create their own node software that abides by the blockchain's protocols they could accept any kind of coin they wished as payment.

```
{
     type: coin,
     data: {
          id: "mono",
          created-by: "system", 
          quantity: 50
     }
}
```

### Certificates

This represents a certificate of ownership and is what allows [Nfts](../../terminology.md#nfts) to be supported on this blockchain, the important thing to know about these is that they can hold multiple owners depending on which part of the certificate you are looking at, and they should only be allowed to be edited or "sent" by the wallet that owns that part of the certificate.

### Creation

this is what is used to "create" something on the blockchain, if this is custom then it can just receive a check for compulsory properties (type) and then assumed to be valid, it's up to whoever is implementing it to figure out how to go from here. The number of coins that can be created is based upon the amount of time since the epoch, where the amount of coins per block will begin at 1000 and will be multiplied by 0.9 and then floored to the nearest integer value for every month that the blockchain exists past the origin time - which will be defined later but basically just represents the moment the blockchain was "started".

TrustElo - this is what is sent from a leader node to a worker node after a block has been added to the blockchain, it can be positive or negative, but has a fixed value of +1 for a valid block and -1 for an invalid block, it is up to the leader node to add this to transactions and that's why they have fixed values, to prevent a leader node from being able to destroy/boost the TrustElo of each node in it's pool if a bad actor gained control of it. In order to send TrustElo&#x20;

### Custom

## Limitations

### A large enough testing base
