# Trading-master
pip install ccxt pandas numpy
import ccxt

exchange = ccxt.binance({
    'apiKey': 'YOUR_API_KEY',
    'secret': 'YOUR_SECRET_KEY',
    'enableRateLimit': True
})
import pandas as pd

symbol = 'BTC/USDT'
timeframe = '1h'  # 1-hour intervals
limit = 500  # Number of data points

ohlcv = exchange.fetch_ohlcv(symbol, timeframe=timeframe, limit=limit)
df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
short_window = 50
long_window = 200

df['SMA50'] = df['close'].rolling(window=short_window).mean()
df['SMA200'] = df['close'].rolling(window=long_window).mean()

def generate_signal(data):
    if data['SMA50'].iloc[-1] > data['SMA200'].iloc[-1] and data['SMA50'].iloc[-2] <= data['SMA200'].iloc[-2]:
        return 'buy'
    elif data['SMA50'].iloc[-1] < data['SMA200'].iloc[-1] and data['SMA50'].iloc[-2] >= data['SMA200'].iloc[-2]:
        return 'sell'
    else:
        return 'hold'

signal = generate_signal(df)
print(f"Current signal: {signal}")
def place_order(signal, symbol, amount):
    try:
        if signal == 'buy':
            order = exchange.create_market_buy_order(symbol, amount)
            print(f"Buy order executed: {order}")
        elif signal == 'sell':
            order = exchange.create_market_sell_order(symbol, amount)
            print(f"Sell order executed: {order}")
        else:
            print("No action taken.")
    except Exception as e:
        print(f"An error occurred: {e}")

amount = 0.001  # Example amount
place_order(signal, symbol, amount)
import time

while True:
    ohlcv = exchange.fetch_ohlcv(symbol, timeframe=timeframe, limit=limit)
    df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')
    df['SMA50'] = df['close'].rolling(window=short_window).mean()
    df['SMA200'] = df['close'].rolling(window=long_window).mean()
    signal = generate_signal(df)
    place_order(signal, symbol, amount)
    time.sleep(3600)  # Wait for 1 hour before next check
