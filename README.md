# polymarket-faa-attribution-02

Forecast Polymarket's FAA outage attribution prices minute by minute, 4 hours
ahead, as five quantiles (0.05, 0.25, 0.5, 0.75, 0.95) per market per minute.
Python 3.13, standard library only, offline, deterministic.

## Method

Persistence with a per-market realized-volatility spread:

- **Median:** the market's last price (persistence). Minute-to-minute drift
  was tested on the practice cases and hurt — prices mean-revert, they do
  not trend — so no drift term is used.
- **Spread:** sigma(h) = sigma_1min * sqrt(h), where sigma_1min is the robust
  1-minute volatility of the last 360 minutes: mean |1-min move| scaled to
  a normal sd (x1.2533), floored at 0.0004 and capped at 0.006. The cap stops
  one jumpy market from getting an absurdly wide spread; the floor keeps
  dead-quiet markets from collapsing to a point.
- q_tau(h) = clamp(median + z_tau * sigma(h), 0, 1), rounded to 6 decimals.

## Validation (out-of-sample, on the 6 practice cases)

Each practice case was truncated 240 minutes before its checkpoint and scored
against the realized next 240 minutes (taken from the following case's file):

- **Forward eval (p00..p04): mean pinball loss 0.3208** vs the house
  baseline's 0.3985 on the same windows (house practice score over all 6
  cases: 0.363589).
- **24-window truncation robustness check:** this method 0.7804 mean /
  0.4466 median vs house 0.8033 / 0.5361.

## Run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `<id>.in` in the input directory, writes `<id>.out` to the output
directory. Runs in ~0.1 s per case (limit 8 s). Two runs are byte-identical.

## License

MIT — see LICENSE.
