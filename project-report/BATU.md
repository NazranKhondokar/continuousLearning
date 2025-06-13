- `/curlable_data/gold_price_trends?target_day=2025-05-21`
```
{
  "_id": "6838deba27566d1031a2eec4",
  "close_price": 3292.39990234375,
  "date": "2025-05-22T17:00:00",
  "retrieved_timestamp": "2025-05-21T20:44:50.431000"
}
```
It works for the below-required JSON.
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
    }
```

- `/curlable_data/sp500_trends?target_day=2025-05-29`
```
{
  "_id": "6838e041969a88fdafc138ea",
  "close_value": 5861.06982421875,
  "date": "2025-05-22T17:30:00",
  "dividends": 0.0,
  "high_value": 5862.4501953125,
  "low_value": 5855.419921875,
  "open_value": 5858.669921875,
  "retrieved_timestamp": "2025-05-29T20:43:21.667000",
  "stock_splits": 0.0,
  "volume": 46956000.0
}
```
It does not work for the below-required JSON because the provided JSON is for a candlestick chart.
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
