---
title: Set Up an Exchange
---

# Adding DigitalBits to your Exchange
This guide will walk you through the integration steps to add DigitalBits to your exchange. This example uses Node.js and the [JS DigitalBits SDK](https://github.com/xdbfoundation/xdb-digitalbits-sdk), but it should be easy to adapt to other languages.

There are many ways to architect an exchange. This guide uses the following design:
 - `issuing account`: One DigitalBits account that holds the majority of customer deposits offline.
 - `base account`: One DigitalBits account that holds a small amount of customer deposits online and is used to payout to withdrawal requests.
 - `customerID`: Each user has a customerID, used to correlate incoming deposits with a particular user's account on the exchange.

The two main integration points to DigitalBits for an exchange are:<br>
1) Listening for deposit transactions from the DigitalBits network<br>
2) Submitting withdrawal transactions to the DigitalBits network

## Setup

### Operational
* *(optional)* Set up [DigitalBits Core](https://developer.digitalbits.io/digitalbits-core/software/admin.html)
* *(optional)* Set up [Frontier](https://developer.digitalbits.io/frontier/reference/index.html)

If your exchange doesn't see a lot of volume, you don't need to set up your own instances of DigitalBits Core and Frontier. Instead, use one of the DigitalBits.io public-facing Frontier servers.
```
  test net: {hostname:'frontier.testnet.digitalbits.io', secure:true, port:443};
  live: {hostname:'frontier.livenet.digitalbits.io', secure:true, port:443};
```

### Issuing account
An issuing account is typically used to keep the bulk of customer funds secure. An issuing account is a DigitalBits account whose secret keys are not on any device that touches the Internet. Transactions are manually initiated by a human and signed locally on the offline machine—a local install of `xdb-digitalbits-sdk` creates a `tx_blob` containing the signed transaction. This `tx_blob` can be transported to a machine connected to the Internet via offline methods (e.g., USB or by hand). This design makes the issuing account secret key much harder to compromise.

### Base account
A base account contains a more limited amount of funds than an issuing account. A base account is a DigitalBits account used on a machine that is connected to the Internet. It handles the day-to-day sending and receiving of digitalbits. The limited amount of funds in a base account restricts loss in the event of a security breach.

### Database
- Need to create a table for pending withdrawals, `DigitalBitsTransactions`.
- Need to create a table to hold the latest cursor position of the deposit stream, `DigitalBitsCursor`.
- Need to add a row to your users table that creates a unique `customerID` for each user.
- Need to populate the customerID row.

```
CREATE TABLE DigitalBitsTransactions (UserID INT, Destination varchar(56), XLMAmount INT, state varchar(8));
CREATE TABLE DigitalBitsCursor (id INT, cursor varchar(50)); // id - AUTO_INCREMENT field
```

Possible values for `DigitalBitsTransactions.state` are "pending", "done", "error".


### Code

Here is a code framework you can use to integrate DigitalBits into your exchange. The following sections describe each step.

For this guide, we use placeholder functions for reading/writing to the exchange database. Each database library connects differently, so we abstract away those details.

```js
// Config your server
var config = {};
config.baseAccount = "your base account address";
config.baseAccountSecret = "your base account secret key";

// You can use DigitalBits.io's instance of Frontier or your own
config.frontier = 'https://frontier.testnet.digitalbits.io';

// Include the JS DigitalBits SDK
// It provides a client-side interface to Frontier
var DigitalBitsSdk = require('digitalbits-sdk');
// uncomment for live network:
// DigitalBitsSdk.Network.usePublicNetwork();

// Initialize the DigitalBits SDK with the Frontier instance
// You want to connect to
var server = new DigitalBitsSdk.Server(config.frontier);

// Get the latest cursor position
var lastToken = latestFromDB("DigitalBitsCursor");

// Listen for payments from where you last stopped
// GET https://frontier.testnet.digitalbits.io/accounts/{config.baseAccount}/payments?cursor={last_token}
let callBuilder = server.payments().forAccount(config.baseAccount);

// If no cursor has been saved yet, don't add cursor parameter
if (lastToken) {
  callBuilder.cursor(lastToken);
}

callBuilder.stream({onmessage: handlePaymentResponse});

// Load the account sequence number from Frontier and return the account
// GET https://frontier.testnet.digitalbits.io/accounts/{config.baseAccount}
server.loadAccount(config.baseAccount)
  .then(function (account) {
    submitPendingTransactions(account);
  })
```

## Listening for deposits
When a user wants to deposit digitalbits in your exchange, instruct them to send XDB to your base account address with the customerID in the memo field of the transaction.

You must listen for payments to the base account and credit any user that sends XDB there. Here's code that listens for these payments:

```js
// Start listening for payments from where you last stopped
var lastToken = latestFromDB("DigitalBitsCursor");

// GET https://frontier.testnet.digitalbits.io/accounts/{config.baseAccount}/payments?cursor={last_token}
let callBuilder = server.payments().forAccount(config.baseAccount);

// If no cursor has been saved yet, don't add cursor parameter
if (lastToken) {
  callBuilder.cursor(lastToken);
}

callBuilder.stream({onmessage: handlePaymentResponse});
```


For every payment received by the base account, you must:<br>
-check the memo field to determine which user sent the deposit.<br>
-record the cursor in the `DigitalBitsCursor` table so you can resume payment processing where you left off.<br>
-credit the user's account in the DB with the number of XDB they sent to deposit.

So, you pass this function as the `onmessage` option when you stream payments:

```js
function handlePaymentResponse(record) {

  // GET https://frontier.testnet.digitalbits.io/transaction/{id of transaction this payment is part of}
  record.transaction()
    .then(function(txn) {
      var customer = txn.memo;

      // If this isn't a payment to the baseAccount, skip
      if (record.to != config.baseAccount) {
        return;
      }
      if (record.asset_type != 'native') {
         // If you are a XDB exchange and the customer sends
         // you a non-native asset, some options for handling it are
         // 1. Trade the asset to native and credit that amount
         // 2. Send it back to the customer  
      } else {
        // Credit the customer in the memo field
        if (checkExists(customer, "ExchangeUsers")) {
          // Update in an atomic transaction
          db.transaction(function() {
            // Store the amount the customer has paid you in your database
            store([record.amount, customer], "DigitalBitsDeposits");
            // Store the cursor in your database
            store(record.paging_token, "DigitalBitsCursor");
          });
        } else {
          // If customer cannot be found, you can raise an error,
          // add them to your customers list and credit them,
          // or do anything else appropriate to your needs
          console.log(customer);
        }
      }
    })
    .catch(function(err) {
      // Process error
    });
}
```


## Submitting withdrawals
When a user requests a lumen withdrawal from your exchange, you must generate a DigitalBits transaction to send them the digitalbits. Here is additional documentation about [Building Transactions](https://developer.digitalbits.io/xdb-digitalbits-base/learn/building-transactions.html).

The function `handleRequestWithdrawal` will queue up a transaction in the exchange's `DigitalBitsTransactions` table whenever a withdrawal is requested.

```js
function handleRequestWithdrawal(userID,amountLumens,destinationAddress) {
  // Update in an atomic transaction
  db.transaction(function() {
    // Read the user's balance from the exchange's database
    var userBalance = getBalance('userID');

    // Check that user has the required digitalbits
    if (amountLumens <= userBalance) {
      // Debit the user's internal lumen balance by the amount of digitalbits they are withdrawing
      store([userID, userBalance - amountLumens], "UserBalances");
      // Save the transaction information in the DigitalBitsTransactions table
      store([userID, destinationAddress, amountLumens, "pending"], "DigitalBitsTransactions");
    } else {
      // If the user doesn't have enough XDB, you can alert them
    }
  });
}
```

Then, you should run `submitPendingTransactions`, which will check `DigitalBitsTransactions` for pending transactions and submit them.

```js
DigitalBitsSdk.Network.useTestNetwork();
// This is the function that handles submitting a single transaction

function submitTransaction(exchangeAccount, destinationAddress, amountLumens) {
  // Update transaction state to sending so it won't be
  // resubmitted in case of the failure.
  updateRecord('sending', "DigitalBitsTransactions");

  // Check to see if the destination address exists
  // GET https://frontier.testnet.digitalbits.io/accounts/{destinationAddress}
  server.loadAccount(destinationAddress)
    // If so, continue by submitting a transaction to the destination
    .then(function(account) {
      var transaction = new DigitalBitsSdk.TransactionBuilder(exchangeAccount)
        .addOperation(DigitalBitsSdk.Operation.payment({
          destination: destinationAddress,
          asset: DigitalBitsSdk.Asset.native(),
          amount: amountLumens
        }))
        // Sign the transaction
        .build();

      transaction.sign(DigitalBitsSdk.Keypair.fromSecret(config.baseAccountSecret));

      // POST https://frontier.testnet.digitalbits.io/transactions
      return server.submitTransaction(transaction);
    })
    //But if the destination doesn't exist...
    .catch(DigitalBitsSdk.NotFoundError, function(err) {
      // create the account and fund it
      var transaction = new DigitalBitsSdk.TransactionBuilder(exchangeAccount)
        .addOperation(DigitalBitsSdk.Operation.createAccount({
          destination: destinationAddress,
          // Creating an account requires funding it with XDB
          startingBalance: amountLumens
        }))
        .build();

      transaction.sign(DigitalBitsSdk.Keypair.fromSecret(config.baseAccountSecret));

      // POST https://frontier.testnet.digitalbits.io/transactions
      return server.submitTransaction(transaction);
    })
    // Submit the transaction created in either case
    .then(function(transactionResult) {
      updateRecord('done', "DigitalBitsTransactions");
    })
    .catch(function(err) {
      // Catch errors, most likely with the network or your transaction
      updateRecord('error', "DigitalBitsTransactions");
    });
}

// This function handles submitting all pending transactions, and calls the previous one
// This function should be run in the background continuously

function submitPendingTransactions(exchangeAccount) {
  // See what transactions in the db are still pending
  // Update in an atomic transaction
  db.transaction(function() {
    var pendingTransactions = querySQL("SELECT * FROM DigitalBitsTransactions WHERE state =`pending`");

    while (pendingTransactions.length > 0) {
      var txn = pendingTransactions.pop();

      // This function is async so it won't block. For simplicity we're using
      // ES7 `await` keyword but you should create a "promise waterfall" so
      // `setTimeout` line below is executed after all transactions are submitted.
      // If you won't do it will be possible to send a transaction twice or more.
      await submitTransaction(exchangeAccount, tx.destinationAddress, tx.amountLumens);
    }

    // Wait 30 seconds and process next batch of transactions.
    setTimeout(function() {
      submitPendingTransactions(sourceAccount);
    }, 30*1000);
  });
}
```

## Going further...
### Federation
The federation protocol allows you to give your users easy addresses—e.g., bob*yourexchange.com—rather than cumbersome raw addresses such as: GCEZWKCA5VLDNRLN3RPRJMRZOX3Z6G5CHCGSNFHEYVXM3XOJMDS674JZ?19327

For more information, check out the [federation guide](./concepts/federation.md).

### Anchor
If you're an exchange, it's easy to become a DigitalBits anchor as well. The integration points are very similar, with the same level of difficulty. Becoming a anchor could potentially expand your business.

To learn more about what it means to be an anchor, see the [anchor guide](./anchor/readme.md).
