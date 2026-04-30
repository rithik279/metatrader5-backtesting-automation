# MT5 Remote Backtest Automation

Automated backtesting system for MetaTrader 5 via remote desktop. Processes optimization result files (.set) through a two-stage backtest-forward workflow, extracts metrics, and logs results to CSV.

## Overview

```
┌─────────────┐     OneDrive      ┌─────────────────┐
│ Local PC    │ ────────────────► │ Remote VPS/RDP  │
│ (Queue .set)│                   │ (runs MT5)      │
└─────────────┘                   └─────────────────┘
                                            │
                                            ▼
                                    results.csv (syncs back)
```

## Architecture

**File-based IPC** — No direct network connection. Local machine drops `.set` files into OneDrive Queue folder, remote VPS picks them up and processes.

**State Machine via Folders:**
- `Queue/` → files waiting to be processed
- `Processing/` → files currently being worked
- `Processed/` → completed files (archived)

**Two-Pass Backtest Flow:**
1. **Backtest Portion**: `FromDate` → `ForwardSplitDate`
2. **Forward Portion**: `ForwardSplitDate` → `ToDate`

Each pass runs MT5 separately, parses HTML report, extracts metrics.

## Folder Structure

```
RD_MT5_Sharing/
├── remote_runner.py       # Main automation script
├── remote_config.json     # Dynamic configuration (symbol, dates, deposit)
├── Queue/                 # Drop .set files here
├── Processing/            # Active work directory
├── Processed/             # Completed runs
├── Results/
│   └── results.csv        # Combined metrics output
└── mt5.ini                # Generated per-run (not committed)
```

## Setup

### 1. VPS/Remote Desktop

```powershell
# Clone or copy repo to VPS
git clone <repo-url>
cd RD_MT5_Sharing

# Install Python (3.8+)
python remote_runner.py
```

### 2. MT5 Configuration

Edit paths in `remote_runner.py`:
```python
MT5_TERMINAL_PATH = r"C:\Program Files\PU Prime MT5 Terminal-1\terminal64.exe"
MT5_DATA_FOLDER_NAME = "CB73EB447A09F27F5775C81FBB987ED5"  # Your terminal hash
EA_NAME = r"Advisors\Archangel_X-v3.4.ex5"  # Your EA path
```

### 3. OneDrive Sync

Ensure `RD_MT5_Sharing` folder syncs between local PC and VPS.

### 4. Local PC

Drop `.set` files into the Queue folder. They sync to VPS automatically.

## Configuration

Edit `remote_config.json` on OneDrive (synced to VPS):

```json
{
  "Symbol": "NAS100ft.s",
  "Deposit": "50000",
  "FromDate": "2025.09.12",
  "SplitDate": "2025.11.12",
  "ToDate": "2025.12.12",
  "ClearResults": false
}
```

`ClearResults: true` clears `results.csv` on next run.

## Output

`results.csv` columns:
- `Timestamp`, `SetFile`, `Pass`
- `BT_*`: Backtest metrics (Profit, Drawdown, DrawdownPct, Trades, WinRate, ProfitFactor, ExpectedPayoff, AvgProfitTrade, AvgLossTrade, MaxConsecLosses)
- `FT_*`: Forward metrics (same fields)

## Requirements

- Python 3.8+
- MetaTrader 5 Terminal (installed on VPS)
- OneDrive sync between local and remote
- MT5 Expert Advisor compiled (.ex5)

## Known Limitations

- No retry logic if MT5 crashes mid-run
- No email/push notifications on failure
- Polling-based (2-second interval)
- HTML parsing brittle to MT5 report format changes

## License

MIT