# AGENTS.md — freqtrade

Crypto trading bot, Python >=3.11. Two installable packages: `freqtrade/` (main) and `ft_client/` (REST client library).

## Install

```bash
pip install -r requirements-dev.txt
pip install -e ft_client/    # must install BEFORE main package
pip install -e .
```

CI uses `uv pip` instead of `pip`. `setup.sh` also prefers `uv` if available.

## Verify

```bash
ruff check .                  # lint
ruff format --check           # format check (line-length=100)
mypy freqtrade scripts tests  # typecheck (ignores errors in tests/*)
pytest --random-order --durations 20 -n auto   # all tests
```

Run a single test: `pytest tests/test_file.py::test_name`

## Test quirks

- `@pytest.mark.longrun` tests are **skipped by default**. Pass `--longrun` to include them (live exchange tests in `tests/exchange_online/`).
- Custom xdist scheduler (`FixtureScheduler`) groups exchange_online tests by exchange ID to avoid rate limits.
- `np.seterr(all="raise")` — numpy errors raise exceptions, not warnings.
- On macOS, `torch` is fully mocked due to compatibility issues.
- Key fixtures in `tests/conftest.py`: `default_conf`, `patch_exchange`, `get_patched_exchange`, `patch_freqtradebot`, `init_persistence` (in-memory SQLite).
- Autouse fixture `backtesting_cleanup` in `tests/optimize/conftest.py` calls `Backtesting.cleanup()` after each test.
- Helper constants: `EXMS = "freqtrade.exchange.exchange.Exchange"`, `TRADE_SIDES = ("long", "short")`.

## Generated files

These scripts must produce no repo changes (CI dirty-checks after running them):

- `python build_helpers/extract_config_json_schema.py` → generates JSON schema
- `python build_helpers/create_command_partials.py` → generates command doc partials
- `python build_helpers/freqtrade_client_version_align.py` → aligns ft_client version with main package

If you modify config schema or CLI commands, re-run the relevant helper.

## Architecture

- **Entry point**: `freqtrade.main:main` (CLI via `freqtrade` command)
- **Core loop**: `freqtrade/worker.py` → `freqtrade/freqtradebot.py`
- **Exchange layer**: `freqtrade/exchange/` (ccxt-based, per-exchange modules)
- **Persistence**: `freqtrade/persistence/` (SQLAlchemy models: Trade, Order, etc.)
- **RPC/API**: `freqtrade/rpc/` (Telegram, FastAPI REST+WebSocket on uvicorn)
- **ML subsystem**: `freqtrade/freqai/` (has its own requirements: `requirements-freqai.txt`)
- **Backtesting/Hyperopt**: `freqtrade/optimize/`
- **Strategy interface**: `freqtrade/strategy/` (`IStrategy`)
- **Templates**: `freqtrade/templates/` (Jinja2-generated strategies, relaxed lint, excluded from coverage)
- **Vendored code**: `freqtrade/vendor/` (ignored by pylint, excluded from coverage)

## Style

- Line length: **100** (not 88)
- 2 blank lines after imports (ruff isort `lines-after-imports = 2`)
- Docstrings: reST format (`:param xxx:`, `:return:`), double quotes
- Codespell ignores: `coo, fo, strat, zar, selectin`
- Pre-commit runs: schema extraction → mypy → ruff → trailing-whitespace → codespell

## Conventions

- **PRs target `develop`**, not `stable`.
- `TA-Lib` C library must be installed system-wide before the Python wrapper works.
- Pyright is configured with `typeCheckingMode = "off"` — mypy is the active type checker.
- Mypy uses the SQLAlchemy plugin (`plugins = ["sqlalchemy.ext.mypy.plugin"]`).

## Memory

All organized learning notes and reference docs go in `memory/`:

| File | Content |
|------|---------|
| `memory/freqtrade-core-concepts.md` | 运行流程、IStrategy 框架、信号机制、回调方法、HyperOpt、数据流 |
| `memory/technical-indicators.md` | RSI、MACD、布林带原理与用法，指标组合思路 |
| `memory/crypto-price-prediction-learning-path.md` | Qlib 加密货币价格预测学习路线 |
| `memory/quant-trade.md` | 量化交易核心算法概览（LightGBM、深度学习、GARCH 等） |

When answering questions about freqtrade concepts, technical indicators, or quantitative trading, reference these files first. All future learning notes should also be placed in `memory/`.
