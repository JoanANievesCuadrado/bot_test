# P2PLNBot
Telegram bot which allows to people to trade using lightning network with other people on telegram, this is an open source project and anyone can create issues, submit a PR, fork it, modify it or create their own bot with the code.

**p2plnbot** is being developed on nodejs and connect with an LND node, after some thinking we couldn't find a good way telegram bot that give this kind of service without being custodial, but we developed a **trust minimized** model for this, we are not custodians of funds of the users, this is why we use hold invoices, the bot only settle seller invoices when each party is ok with it and at that same times the bot pay the buyer's invoice.

## Creating a sell order
1. Alice tells to the bot that she wants to sell 5000 sats by **n** fiat amount.
2. The bot send to alice a hold-invoice, Alice has to pay a hold-invoice of 5000 sats to the bot, this invoice is "pending", the money is not accepted or rejected by the bot at that moment.
3. After the bot detects that Alice paid the invoice, the order is published on the channel.
4. Bob accepts the order for 5000 sats sending a new lightning invoice to the bot.
5. The bot put in contact Bob and Alice.
6. Bob sends the fiat money via bank transfer.
7. When Alice confirmed that she received the money, the bot settles Alice's initial invoice and pays Bob's invoice.
8. If Alice does not confirm the operation that she received the payment in certain amount of time (initially we set this in two hours but this can be changed), the bot cancels the hold-invoice and closes the order, before the time expires Bob will be notified that Alice is not responding and Bob can start a dispute.

## Creating a buy order
1. Alice wants to buy 5000 sats.
2. Alice publishes a buy order of 5000 sats with **n** fiat amount, Alice does not have satoshis, but she has fiat, Alice sends an invoice to the bot, the bot saves it when she creates the order.
3. The bot shows the order in the public group.
4. Bob takes the order, the bot sends him a hold-invoice for 5000.
5. Bob pays the invoice.
6. The bot tells Alice that Bob has already made the payment, she can now send the fiat to Bob.
7. When Bob confirms that he received the fiat, the bot settles Bob's invoice and pays Alice's invoice.
8. If Bob does not confirm the operation that he received the payment in certain amount of time (initially we set this in two hours but this can be changed), the bot cancels the hold-invoice and closes the order, before the time expires Alice will be notified that Bob is not responding and Alice can start a dispute.

## Cooperative close
After a user creates a new order and before another user take it, the user can cancel the order, but in some cases users can need to cancel the order, it shouldn't be unilateral.

Only if both parties cancel cooperatively the order is canceled and the seller sats are returned.

If users have a disagreement on canceling or going forward the can open a dispute.

## Disputes
Both parties can start a dispute at any moment, after a dispute is started a human will be notified with all the information, this human will contact both parties to evaluate the situation and take a decision.

After a user starts a dispute, both parties will have increased by 1 their own `dispute` field in database and after **4** disputes users will be banned from using the bot.

## Incentive to release funds
A seller that didn't release funds to the buyer can't open or take another order from the bot and probably will be involved in a dispute from the buyer damaging his/her reputation

# Instalation
You will need to create an `.env` file on the root dir, you can use a sample env file called `.env.sample` on the root dir, an easy way of doing this is just to copy the sample file to `.env`.

```
cp .env-sample .env
```

## MongoDB
You will need to have [mongo](https://www.mongodb.com) installed and fill the mongo variables on the .env file, those that stats with `DB_`.

## Telegram
You will need a telegram bot api key (`BOT_TOKEN`), find out more about it [here](https://core.telegram.org/bots/).

## Lightning Network
You will need a lightning network node, for this bot we use [LND](https://github.com/lightningnetwork/lnd/).

To connect with a lnd node we need to set 3 variables in the `.env` file,

*LND_CERT_BASE64:* LND node TLS certificate on base64 format, you can get it with `base64 ~/.lnd/tls.cert | tr -d '\n'` on the lnd node.

*LND_MACAROON_BASE64:* Macaroon file on base64 format, the macaroon file contains permission for doing actions on the lnd node, you can get it with `base64 ~/.lnd/data/chain/bitcoin/mainnet/admin.macaroon | tr -d '\n'`.

*LND_GRPC_HOST:* IP address or domain name from the LND node and the port separated by colon (`:`), example: `192.168.0.2:10009`.

To install just run:
```
$ git clone https://github.com/grunch/p2plnbot.git
$ cd p2plnbot
$ npm install
```
# Running it
```
$ npm start
```
# Testing
```
$ npm test
```