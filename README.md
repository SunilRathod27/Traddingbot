# Binance Trading Bot


> Automated Binance trading bot with trailing buy/sell strategy

---


This is a test project. I am just testing my code.

## How it works

### Trailing Buy/Sell Bot

This bot is using the concept of trailing buy/sell order which allows following the price fall/rise.

> Trailing Stop Orders
> About Trailing Stop Orders Concept you can find at [Binance Official document](https://www.binance.com/en/support/faq/360042299292)
>
> TL;DR
> Place orders at a fixed value or percentage when the price changes. Using this feature you can buy at the lowest possible price when buying down and sell at the highest possible price when selling up.

- The bot can monitor multiple symbols. All symbols will be monitored per second.
- The bot is using MongoDB to provide a persistence database. However, it does not use the latest MongoDB to support Raspberry Pi 32bit. Used MongoDB version
  is 3.2.20, which is provided by [apcheamitru](https://hub.docker.com/r/apcheamitru/arm32v7-mongo).
- The bot is tested/working with Linux and Raspberry Pi 4 32bit. Other platforms are not tested.

#### Buy Signal

The bot will continuously monitor the lowest value for the period of the candles. Once the current price reaches the lowest price, then the bot will place a STOP-LOSS-LIMIT order to buy. If the current price continuously falls, then the bot will cancel the previous order and re-place the new STOP-LOSS-LIMIT order with the new price.

- The bot will not place a buy order if has enough coin (typically over $10 worth) to sell when reaches the trigger price for selling.

##### Buy Scenario

Let say, if the buy configurations are set as below:

- Maximum purchase amount: $50
- Trigger percentage: 1.005 (0.5%)
- Stop price percentage: 1.01 (1.0%)
- Limit price percentage: 1.011 (1.1%)

And the market is as below:

- Current price: $101
- Lowest price: $100
- Trigger price: $100.5

Then the bot will not place an order because the trigger price ($100.5) is less than the current price ($101).

In the next tick, the market changes as below:

- Current price: $100
- Lowest price: $100
- Trigger price: $100.5

The bot will place new STOP-LOSS-LIMIT order for buying because the current price ($100) is less than the trigger price ($100.5). For the simple calculation, I do not take an account for the commission. In real trading, the quantity may be different. The new buy order will be placed as below:

- Stop price: $100 \* 1.01 = $101
- Limit price: $100 \* 1.011 = $101.1
- Quantity: 0.49

In the next tick, the market changes as below:

- Current price: $99
- Current limit price: $99 \* 1.011 = 100.089
- Open order stop price: $101

As the open order's stop price ($101) is higher than the current limit price ($100.089), the bot will cancel the open order and place new STOP-LOSS-LIMIT order as below:

- Stop price: $99 \* 1.01 = $99.99
- Limit price: $99 \* 1.011 = $100.089
- Quantity: 0.49

If the price continuously falls, then the new buy order will be placed with the new price.

And if the market changes as below in the next tick:

- Current price: $100

Then the current price reaches the stop price ($99.99); hence, the order will be executed with the limit price ($100.089).

### Sell Signal

If there is enough balance for selling and the last buy price is recorded in the bot, then the bot will start monitoring the sell signal. Once the current price reaches the trigger price, then the bot will place a STOP-LOSS-LIMIT order to sell. If the current price continuously rises, then the bot will cancel the previous order and re-place the new STOP-LOSS-LIMIT order with the new price.

- If the coin is worth less than typically $10 (minimum notional value), then the bot will remove the last buy price because Binance does not allow to place an order of less than $10.
- If the bot does not have a record for the last buy price, the bot will not sell the coin.

#### Sell Scenario

Let say, if the sell configurations are set as below:

- Trigger percentage: 1.05 (5.0%)
- Stop price percentage: 0.98 (-2.0%)
- Limit price percentage: 0.979 (-2.1%)

And the market is as below:

- Coin owned: 0.5
- Current price: $100
- Last buy price: $100
- Trigger price: $100 \* 1.05 = $105

Then the bot will not place an order because the trigger price ($105) is higher than the current price ($100).

If the price is continuously falling, then the bot will keep monitoring until the price reaches the trigger price.

In the next tick, the market changes as below:

- Current price: $105
- Trigger price: $105

The bot will place new STOP-LOSS-LIMIT order for selling because the current price ($105) is higher or equal than the trigger price ($105). For the simple calculation, I do not take an account for the commission. In real trading, the quantity may be different. The new sell order will be placed as below:

- Stop price: $105 \* 0.98 = $102.9
- Limit price: $105 \* 0.979 = $102.795
- Quantity: 0.5

In the next tick, the market changes as below:

- Current price: $106
- Current limit price: $103.774
- Open order stop price: $102.29

As the open order's stop price ($102.29) is less than the current limit price ($103.774), the bot will cancel the open order and place new STOP-LOSS-LIMIT order as below:

- Stop price: $106 \* 0.98 = $103.88
- Limit price: $106 \* 0.979 = $103.774
- Quantity: 0.5

If the price continuously rises, then the new sell order will be placed with the new price.

And if the market changes as below in the next tick:

- Current price: $103

The the current price reaches the stop price ($103.88); hence, the order will be executed with the limit price ($103.774).

#### Sell Stop-Loss Scenario

Let say, if the sell Stop-Loss configurations are set as below:

- Max loss percentage: 0.90
- Temporary disable for buying (minutes): 60

And the market is as below:

- Current price: $95
- Last buy price: $100
- Stop-Loss price: $90

Then the bot will not place a Stop-Loss order because the Stop-Loss price ($90) is less than the current price ($95).

If the price is continuously falling, then the bot will keep monitoring until the price reaches the Stop-Loss price.

In the next tick, the market changes as below:

- Current price: $90
- Stop-Loss price: $90

The bot will place new MARKET order for selling because the current price ($90) is less or equal than the Stop-Loss price ($90). In real trading, the quantity may be different.

The bot will also set the symbol to be temporarily disabled for 60 minutes to avoid buying/sell continuously. In the frontend, the action will display the pause icon and how long left to be enabled again. The symbol can be enabled by clicking the play icon.

### [Features](https://github.com/chrisleekr/binance-trading-bot/wiki/Features)

- Manual trade
- Convert small balances to BNB
- Trade all symbols
- Monitoring multiple coins simultaneously
- Stop-Loss
- Restrict buying with ATH price

### Frontend + WebSocket

React.js based frontend communicating via Web Socket:

- List monitoring coins with buy/sell signals/open orders
- View account balances
- Manage global/symbol settings
- Delete caches that are not monitored
- Link to public URL
- Support Add to Home Screen

## Environment Parameters

Use environment parameters to adjust parameters. Check `/config/custom-environment-variables.json` to see list of available environment parameters.

Or use the frontend to adjust configurations after launching the application.

## How to use

1. Create `.env` file based on `.env.dist`.

   | Environment Key                | Description                                                               | Sample Value                                                                                        |
   | ------------------------------ | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
   | BINANCE_LIVE_API_KEY           | Binance API key for live                                                  | (from [Binance](https://binance.zendesk.com/hc/en-us/articles/360002502072-How-to-create-API))      |
   | BINANCE_LIVE_SECRET_KEY        | Binance API secret for live                                               | (from [Binance](https://binance.zendesk.com/hc/en-us/articles/360002502072-How-to-create-API))      |
   | BINANCE_TEST_API_KEY           | Binance API key for test                                                  | (from [Binance Spot Test Network](https://testnet.binance.vision/))                                 |
   | BINANCE_TEST_SECRET_KEY        | Binance API secret for test                                               | (from [Binance Spot Test Network](https://testnet.binance.vision/))                                 |
   | BINANCE_SLACK_ENABLED          | Slack enable/disable                                                      | true                                                                                                |
   | BINANCE_SLACK_WEBHOOK_URL      | Slack webhook URL                                                         | (from [Slack](https://slack.com/intl/en-au/help/articles/115005265063-Incoming-webhooks-for-Slack)) |
   | BINANCE_SLACK_CHANNEL          | Slack channel                                                             | "#binance"                                                                                          |
   | BINANCE_SLACK_USERNAME         | Slack username                                                            | Chris                                                                                               |
   | BINANCE_LOCAL_TUNNEL_ENABLED   | Enable/Disable [local tunnel](https://github.com/localtunnel/localtunnel) | true                                                                                                |
   | BINANCE_LOCAL_TUNNEL_SUBDOMAIN | Local tunnel public URL subdomain                                         | binance                                                                                             |

   *A local tunnel makes the bot accessible from the outside. Please set the subdomain of the local tunnel as a subdomain that only you can remember.*


2. Launch/Update the bot with docker-compose

   Pull latest code first:

   ```bash
   git pull
   ```

   If want production/live mode, then use the latest build image from DockerHub:

   ```bash
   docker-compose -f docker-compose.server.yml pull
   docker-compose -f docker-compose.server.yml up -d
   ```

   Or if using Raspberry Pi 4 32bit, must build again for Raspberry Pi:

   ```bash
   npm run docker:build
   docker-compose -f docker-compose.rpi.yml up -d
   ```

   Or if want development/test mode, then run below commands:

   ```bash
   docker-compose up -d --build
   ```

3. Open browser `http://0.0.0.0:8080` to see the frontend

   - When launching the application, it will notify public URL to the Slack.
   - If you have any issue with the bot, you can check the log to find out what happened with the bot. Please take a look [Troubleshooting](https://github.com/chrisleekr/binance-trading-bot/wiki/Troubleshooting)

### Install via Stackfile

1. In [Portainer](https://www.portainer.io/) create new Stack

2. Copy content of `docker-stack.yml` or upload the file

3. Set environment keys for `binance-bot` in the `docker-stack.yml`

4. Launch and open browser `http://0.0.0.0:8080` to see the frontend

