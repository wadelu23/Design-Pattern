# Strategy Pattern
> 以介面定義策略
> 
> 將策略實體傳遞給其他物件，根據策略執行動作
> 
> 可以靈活替換以及擴展策略

筆記程式碼範例來源

[Solving A Common Issue With The Strategy Pattern // In Python](https://www.youtube.com/watch?v=UPBOOIRrl40&ab_channel=ArjanCodes)

[The code](https://github.com/ArjanCodes/2021-strategy-parameters)

---

- [Strategy Pattern](#strategy-pattern)
  - [流程概述](#流程概述)
  - [階段](#階段)
    - [原版 ( 0-before.py )](#原版--0-beforepy-)
    - [改由 kwargs 傳入 ( 1_with_keyword_args.py )](#改由-kwargs-傳入--1_with_keyword_argspy-)
    - [定義 Parameters ( 2_with_parameters_class.py )](#定義-parameters--2_with_parameters_classpy-)
    - [定義 init args ( 3_with_init_args.py )](#定義-init-args--3_with_init_argspy-)

---

## 流程概述
以介面( TradingStrategy )定義策略

將策略傳遞給交易機器人，根據策略買賣

(此處簡易貼出部分程式碼說明，完整程式碼請看各檔案)

---

## 階段

### 原版 ( 0-before.py )

建立`MinMaxTradingStrategy`策略

但如果需更改min或max，需進入該物件修改

```python
# create the trading strategy
trading_strategy = MinMaxTradingStrategy()

# create the trading bot and run the bot once
bot = TradingBot(exchange, trading_strategy)
bot.run("BTC/USD")
```

### 改由 kwargs 傳入 ( 1_with_keyword_args.py )

傳入 `min_price`, `max_price` 改變策略條件

但如果替換策略

需要傳入參數的Key值必須再深入才能得知

```python
class TradingBot:
    """Trading bot that connects to a crypto exchange and performs trades."""

    def __init__(self, exchange: Exchange, trading_strategy: TradingStrategy) -> None:
        self.exchange = exchange
        self.trading_strategy = trading_strategy

    def run(self, symbol: str):
        """Run the trading bot once for a particular symbol, with a given strategy."""
        prices = self.exchange.get_market_data(symbol)
        should_buy = self.trading_strategy.should_buy(prices, min_price=31000)
        should_sell = self.trading_strategy.should_sell(prices, max_price=33000)
        if should_buy:
            self.exchange.buy(symbol, 10)
        elif should_sell:
            self.exchange.sell(symbol, 10)
        else:
            print(f"No action needed for {symbol}.")
```

### 定義 Parameters ( 2_with_parameters_class.py )

定義需傳入參數規格與預設值(`StrategyParameters`)，能更清楚快速了解

但`StrategyParameters`中，包含多餘參數

如`MinMaxTradingStrategy`中，其實不需要`window_size`這參數

```python
@dataclass
class StrategyParameters:
    """Class representing the union of possible parameters for the different strategies."""

    window_size: int = 3
    min_price: float = 32000.0
    max_price: float = 33000.0

class MinMaxTradingStrategy(TradingStrategy):
    """Trading strategy based on price minima and maxima."""

    def should_buy(self, prices: List[float], params: StrategyParameters) -> bool:
        return prices[-1] < params.min_price

    def should_sell(self, prices: List[float], params: StrategyParameters) -> bool:
        return prices[-1] > params.max_price
```

且此時替換策略或修改條件，都需要修改`TradingBot`

不符單一職責原則（Single responsibility principle）

```python
# class TradingBot
def run(self, symbol: str):
        """Run the trading bot once for a particular symbol, with a given strategy."""
        prices = self.exchange.get_market_data(symbol)
        should_buy = self.trading_strategy.should_buy(
            # 如修改最低價格或換成傳遞window_size
            prices, StrategyParameters(min_price=31000)
        )
        should_sell = self.trading_strategy.should_sell(
            # 如修改最高價格或換成傳遞window_size
            prices, StrategyParameters(max_price=33000)
        )
```

### 定義 init args ( 3_with_init_args.py )

改由建立策略實體時需要初始參數

```python
@dataclass
class MinMaxTradingStrategy(TradingStrategy):
    """Trading strategy based on price minima and maxima."""

    min_price: float = 32000.0
    max_price: float = 33000.0

    def should_buy(self, prices: List[float]) -> bool:
        # buy if it's below the min_price
        return prices[-1] < self.min_price

    def should_sell(self, prices: List[float]) -> bool:
        # sell if it's above the max_price
        return prices[-1] > self.max_price


@dataclass
class AverageTradingStrategy(TradingStrategy):
    """Trading strategy based on price averages."""

    window_size: int = 3

    def should_buy(self, prices: List[float]) -> bool:

        list_window = prices[-int(self.window_size) :]
        return prices[-1] < statistics.mean(list_window)

    def should_sell(self, prices: List[float]) -> bool:
        list_window = prices[-int(self.window_size) :]
        return prices[-1] > statistics.mean(list_window)
```

使用該物件時，便能快速了解可調參數，不用再追其他class

```python
# main()

# 省略...

# create the trading strategy
trading_strategy = MinMaxTradingStrategy(min_price=31000, max_price=34000)
```

`class TradingBot` 內也更乾淨

不需知道職責以外的細節

且改變策略時，也不需修改`class TradingBot`了

```python
# class TradingBot

def run(self, symbol: str):
        """Run the trading bot once for a particular symbol, with a given strategy."""
        prices = self.exchange.get_market_data(symbol)
        should_buy = self.trading_strategy.should_buy(prices)
        should_sell = self.trading_strategy.should_sell(prices)
```