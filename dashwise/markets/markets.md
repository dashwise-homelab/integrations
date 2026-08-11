# Markets

Markets displays a glanceable quote for a stock or other Yahoo Finance-supported symbol, including the current price, daily change, direction, and a market icon.

## Files

- `integration.yaml` contains the Yahoo Finance request, response mapping, computed quote fields, and glanceable configuration.
- `markets.md` documents the integration for contributors and users.

## Configuration

- `STOCK` — required symbol, such as `AAPL`, `TSLA`, or `BTC-USD`.
- `RANGE` — optional Yahoo Finance chart range; defaults to `1mo`.
- `INTERVAL` — optional quote interval; defaults to `1d`.
- `POLLING_INTERVAL` — optional display polling interval in seconds.
