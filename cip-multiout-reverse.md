    CIP: ?
    Title: Multi-Out Reverse Lookup
    Author: harry1453
    Comments-Summary: TBC
    Comments-URI: TBC
    Status: Draft
    Type: Enhancement
    Created: 2019-01-07

## Motivation

Multi-out transactions were implemented in BRS 2.2.0 as described in CIP 4. However, users have since been frustrated that multi-out transactions do not appear in their list of recent transactions - in fact, there is absolutely no way to detect when you have been paid through a multi-out transaction other than noticing your balance change. This is unacceptable in a currency where automated systems (such as exchanges) may need to detect when money has been transferred to them.

It is essential that this is accessible through the decentralized wallet, not through a third-party database of multi-out transactions.

## Abstract

The node should keep a table in its database with two fields, one of which is the account ID that was paid, and the other of which is the transaction ID of the Multi-Out (Same) transaction in which they were paid.

When a user requests to lookup their recent transactions, the node should, in addition to selecting transactions from the transactions table:

* Select from the multiout reverse table rows matching the user's account ID

* Lookup the multi-out transaction that the row points to

* Lookup the amount that was transferred to the user and include it in the list of transactions returned to the user

## Specification

This CIP will discuss Multi-Out (Same) transactions purely from a database storage perspective - I acknowledge that this is not the only advantage of multi-out transactions.

### Multi-Out Transactions (As defined in CIP 4)

Currently, a full multi-out transaction can have up to 64 recipients. The format is:

* 1 byte - number of recipients (1-64)

* Then up to 64 (depending on previous byte) iterations of

  * 8 bytes (unsigned long) - Account ID to send to
  
  * 8 bytes (unsigned long) - Amount to send to that account
  
Resulting in a maximum storage space consumed of `176+1+64*(8+8) = 1201` bytes. (176 bytes for transaction headers)

Without CIP 4, the same would be achieved in 64 regular transactions, each of which are 176 bytes long. So, the storage space consumed by this operation would be `64*176 = 11,264 bytes`.

Therefore, a Multi-Out transaction represents a storage consumption decrease by `(11264-1201)/11264 = 89.3%`, which is pretty efficient.

However, this comes with the drawback of not being able to see when you have been paid as described in the Abstract above.

### Multi-Out Same Transactions (As defined in CIP 4)

Currently, a full multi-out same transaction can have up to 128 recipients. The format is:

* 1 byte - number of recipients (1-128)

* Then up to 64 (depending on previous byte) iterations of

  * 8 bytes (unsigned long) - Account ID to send to
  
Resulting in a maximum storage space consumed of `176+1+128*(8) = 1201` bytes. (176 bytes for transaction headers)

Without CIP 4, the same would be achieved in 128 regular transactions, each of which are 176 bytes long. So, the storage space consumed by this operation would be `128*176 = 22,528 bytes`.

Therefore, a Multi-Out Same transaction represents a storage consumption decrease by `(22528-1201)/22528 = 94.7%`, which is very efficient.

However, this comes with the drawback of not being able to see when you have been paid as described in the Abstract above.

### Objectives:

1. Don't lose the transaction fee, block slot and speed reduction of Multi-Out (Same) transactions.

2. Allow a user to see when they have been paid via Multi-Out (Same) transactions.

3. Don't place the wallet database under extreme strain by looking up all Multi-Out (Same) transactions to check whether one involved the requested account, every time a request is made

4. Preserve, to the greatest possible extent, the storage consumption benefits, whilst fully implementing #2 and #3.

5. Implement all of this in a way that does not require the network to fork.

### Proposed Solution:

1. Implement a background task that will scan existing blocks and, when new blocks come in, scan them, and analyze all Multi-Out (Same) transactions. When this task finds a Multi-Out (Same) transaction it should add each recipient account ID and the transaction ID of the current transaction to a new database table, which is defined as follows:

| Column Name | Data Type | Description | Example Value |
|-|-|-|-|
| recipient | BIGINT (64-bit unsigned long) | The account ID of the recipient of a Multi-Out (Same) transaction | 7009665667967103287 |
| transaction | BIGINT (64-bit unsigned long) | The transaction ID of the Multi-Out (Same) transaction in which this account was paid | 3631659512270044993 |

For example, if the service process a Multi-Out with transaction ID `TransactionA` and accounts `AccountA`, `AccountB` and `AccountC` are paid an amount, and then processes a Multi-Out Same with transaction ID `TransactionB` where accounts `AccountA` and `AccountD` are paid, it would add the rows:

| recipient | transaction |
|-|-|
| AccountA | TransactionA |
| AccountB | TransactionA |
| AccountC | TransactionA |
| AccountA | TransactionB |
| AccountD | TransactionB |

2. When a user requests their transaction list, search the table for all rows where `recipient` is equal to the user's account ID, which would in this example for `AccountA` return:

| recipient | transaction |
|-|-|
| AccountA | TransactionA |
| AccountA | TransactionB |

3. Lookup the transactions found and add them to the list of regular transactions obtained from the transactions table in the database.

4. When displaying the list of transactions, display the incoming amount as the amount that `AccountA` was paid in the Multi-Out (Same) transaction

### Impact on database storage

Each row in the new table consumes 16 bytes, and there is one per recipient per transaction.

Therefore the maximum amount of additional space consumed per Multi-Out transaction is `16*64 = 1024` bytes, which is `1024/1201 = 85%` more than was previously consumed per Multi-Out transaction.

Additionally, the maximum amount of additional space consumed per Multi-Out Same transaction is `16*128 = 2048` bytes, which is `2048/1201 = 171%` more than was previously consumed per Multi-Out transaction.

Additionally, the new space consumption decrease (compared to not having Multi-Out transactions) is now `(11264-(1201+1024))/11264 = 80.2%` (down from `89.3%` without the new table).

Additionally, the new space consumption decrease (compared to not having Multi-Out Same transactions) is now `(22528-(1201+2048))/22528 = 85.6%` (down from `94.7%` without the new table).

Therefore, I am satisfied that objective #4 is fulfilled, as we are taking a relatively minor hit in storage consumption for the added benefit of having multi-out reverse lookup (especially considering that Multi-Outs are used considerably more than Multi-Out Sames as the network pools are constantly using them, and they are affected less by this change)

## Compatibility

This will affect users of the `getAccountTransactions` and `getAccountTransactionIDs` API calls, so we should add a boolean flag to the request parameters to "Include Incoming Multi-Outs" which will default to false, as otherwise clients using the API that have not update will be confused by the fact that the account is not the recipient of the transaction and by the fact that the transaction amount is not the amount sent to the account.

This should otherwise be implemented within the node and P2P should not be touched, and as long as the Web UI is upgraded there should be no further compatibility issues.

## References

* [CIP 4](cip-0004.md)

## Copyright

This document is placed in the public domain.
