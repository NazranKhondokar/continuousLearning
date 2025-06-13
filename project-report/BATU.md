- `/curlable_data/gold_price_trends?target_day=2025-05-21`

It works for
```
"gold": {
      "symbol": "GOLD",
      "enabled": true,
      "currentPrice": 2048.50,
      "color": "#FFD700",
      "lineData": [
        { "timestamp": 1640908800000, "price": 1980.20 },
        { "timestamp": 1641513600000, "price": 2048.50 }
      ]
    },
```

- `/curlable_data/sp500_trends?target_day=2025-05-29`

It does not work for
```
"sp500": {
          "symbol": "S&P 500",
          "enabled": true,
          "currentPrice": 4325.18,
          "color": "#FF6B35",
          "lineData": [
            { "timestamp": 1640908800000, "price": 4180.25 },
            { "timestamp": 1641513600000, "price": 4325.18 }
          ]
        }
```
