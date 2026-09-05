# Architecture

The system uses synchronized folders as a durable queue between a local workstation and a Windows VPS running MetaTrader 5.

1. A parameter file enters `Queue/`.
2. The runner moves it to `Processing/`.
3. MT5 runs the backtest window and writes an HTML report.
4. MT5 runs the forward window and writes a second report.
5. The runner parses both reports and appends normalized metrics to `Results/results.csv`.
6. The parameter file moves to `Processed/`.

This design avoids exposing a network service, but it depends on filesystem synchronization and polling.
