# PWACL
Title: Building an automated index for your binance account in Python

Date: 04.11.2020 - 18.12.2020

Authors: Tim Graf, Marvin Scherer, Henri de Montpellier

Description:
This is the official code for a project in the course "Programming with Advanced Computer Languages" at the University of St. Gallen. The aim is to build a trading bot which takes your binance account and rebalances it for a given frequency, all automatically. 



# Install the python-binance library
pip install python-binance

# Securing your API keys for Mac users
export binance_api="your_api_key_here"
export binance_secret="your_api_secret_here"

# Retrieve your balance account
import os

from binance.client import Client

# init
api_key = os.environ.get('binance_api')
api_secret = os.environ.get('binance_secret')
client = Client(api_key, api_secret)
client.API_URL = 'https://testnet.binance.vision/api'

# get balances for all assets & some account information
print(client.get_account())

# get balance for a specific asset only (BTC)
print(client.get_asset_balance(asset='BTC'))

# get balances for futures account
print(client.futures_account_balance())

# get balances for margin account
print(client.get_margin_account())

# get latest price from Binance API
btc_price = client.get_symbol_ticker(symbol="BTCUSDT")
# print full output (dictionary)
print(btc_price)

print(btc_price["price"])

# Give a buy order for eth example

# create a real order if the test orders did not raise an exception

try:
    buy_limit = client.create_order(
        symbol='ETHUSDT',
        side='BUY',
        type='LIMIT',
        timeInForce='GTC',
        quantity=100,
        price=200)

except BinanceAPIException as e:
    # error handling goes here
    print(e)
except BinanceOrderException as e:
    # error handling goes here
    print(e)

# Some useful help functions

order_limit_buy()
order_limit_sell()
order_market_buy()
order_market_sell()
order_oco_buy()
order_ocosell()

# Implement a stop loss

try:
    order = client.create_oco_order(
        symbol='ETHUSDT',
        side='SELL',
        quantity=100,
        price=250,
        stopPrice=150,
        stopLimitPrice=150,
        stopLimitTimeInForce='GTC')

except BinanceAPIException as e:
    # error handling goes here
    print(e)
except BinanceOrderException as e:
    # error handling goes here
    print(e)
    
   # Execute a trade on ETH when BTC hits a certain price

import os
from time import sleep

from binance.client import Client
from binance.exceptions import BinanceAPIException, BinanceOrderException
from binance.websockets import BinanceSocketManager
from twisted.internet import reactor

# init
api_key = os.environ.get('binance_api')
api_secret = os.environ.get('binance_secret')
client = Client(api_key, api_secret)
price = {'BTCUSDT': None, 'error':False}

def btc_pairs_trade(msg):
    ''' define how to process incoming WebSocket messages '''
    if msg['e'] != 'error':
        price['BTCUSDT'] = float(msg['c'])
    else:
        price['error']:True
        
        bsm = BinanceSocketManager(client)
conn_key = bsm.start_symbol_ticker_socket('BTCUSDT', btc_pairs_trade)
bsm.start()

while not price['BTCUSDT']:
    # wait for WebSocket to start streaming data
    sleep(0.1)
    
while True:
    # error check to make sure WebSocket is working
    if price['error']:
        # stop and restart socket
        bsm.stop_socket(conn_key)
        bsm.start()
        price['error'] = False

    else:
        if price['BTCUSDT'] > 10000:
            try:
                order = client.order_market_buy(symbol='ETHUSDT', quantity=100)
                break
            except BinanceAPIException as e:
                # error handling goes here
                print(e)
            except BinanceOrderException as e:
                # error handling goes here
                print(e)
    sleep(0.1)
    
    bsm.stop_socket(conn_key)
reactor.stop()

# Source: https://algotrading101.com/learn/binance-python-api-guide/ 
