Script to display Bitcoin price with some price history. Red green color indicator. Refresh every 1 minute. Used to display on small terminal screen.

```python
import requests
import curses
import time
from datetime import datetime, timedelta

# Function to fetch current Bitcoin price
def get_current_bitcoin_price():
    url = 'https://api.coindesk.com/v1/bpi/currentprice/BTC.json'
    response = requests.get(url)
    data = response.json()
    price = data['bpi']['USD']['rate_float']
    return price

# Function to fetch historical Bitcoin price
def get_historical_bitcoin_price(timestamp):
    url = f'https://api.coingecko.com/api/v3/coins/bitcoin/history'
    params = {'date': timestamp.strftime('%d-%m-%Y')}
    response = requests.get(url, params=params)
    data = response.json()
    price = data['market_data']['current_price']['usd']
    return price

# Function to calculate the price changes for different timeframes
def get_price_changes():
    current_price = get_current_bitcoin_price()
    now = datetime.now()
    prices = {}

    # Calculate timestamps for the required timeframes
    timestamps = {
        '15m': now - timedelta(minutes=15),
        '1h': now - timedelta(hours=1),
        '4h': now - timedelta(hours=4),
        '1d': now - timedelta(days=1)
    }

    # Fetch historical prices for the timestamps
    for label, timestamp in timestamps.items():
        prices[label] = get_historical_bitcoin_price(timestamp)

    return current_price, prices

# Function to display Bitcoin price in a terminal
def display_price(stdscr):
    curses.curs_set(0)  # Hide cursor
    stdscr.nodelay(1)   # Non-blocking input
    stdscr.timeout(60000)  # Refresh every 60 seconds

    # Initialize color pairs
    curses.start_color()
    curses.init_pair(1, curses.COLOR_GREEN, curses.COLOR_BLACK)
    curses.init_pair(2, curses.COLOR_RED, curses.COLOR_BLACK)

    last_price = None

    while True:
        stdscr.clear()

        current_price, prices = get_price_changes()
        now = datetime.now()

        if last_price is None:
            color = curses.color_pair(1)
        else:
            color = curses.color_pair(1) if current_price >= last_price else curses.color_pair(2)

        stdscr.addstr(2, 2, now.strftime("%Y-%m-%d %H:%M"), curses.A_DIM)
        stdscr.addstr(4, 2, "")
        stdscr.addstr(6, 2, "Bitcoin Price", curses.A_BOLD)
        stdscr.addstr(8, 2, f"Current: ${current_price:,.2f}", color)
        stdscr.addstr(10, 2, "")
        stdscr.addstr(12, 2, "Price History", curses.A_BOLD)
        y_offset = 14
        for label, historical_price in prices.items():
            change = current_price - historical_price
            change_percent = (change / historical_price) * 100
            change_color = curses.color_pair(1) if change >= 0 else curses.color_pair(2)
            stdscr.addstr(y_offset, 2, f"{label}: ${historical_price:,.2f} ({change:+.2f}, {change_percent:+.2f}%)", change_color)
            y_offset += 2

        last_price = current_price

        stdscr.refresh()

        key = stdscr.getch()
        if key == ord('q'):
            break

        time.sleep(60)

if __name__ == "__main__":
    curses.wrapper(display_price)
```