# GoMining Flywheel — GMT Lock Calculator

A single-page, no-build calculator for [GoMining](https://gomining.com) miners running the **Flywheel Strategy** (originally written up by u/DracoF on [r/gomining](https://www.reddit.com/r/gomining/)), extended to a "125% OPEX coverage" target: how much GMT do you need locked so the dividend yield alone covers your weekly electricity + service costs?

**[Live site →](https://jdobbsclt.github.io/gomining-flywheel-calculator/)**

## What it does

Every input is editable — total hashpower, efficiency (W/TH), GMT price, power rate, and all four independent maintenance-discount components (token, service streak, VIP tier, mining mode) — and results update live:

- Weekly OPEX in USD and GMT
- Estimated locked/liquid GMT from your "days covered" figures
- Weekly dividend at an assumed lock APR
- **Required locked GMT** to hit a chosen OPEX-coverage target (default 125%)
- **Breakeven APR** — the yield at which your current locked GMT already meets the target
- A scenario table comparing required GMT across several APR reference points

## Why the formulas are trustworthy (and where they came from)

The maintenance-cost formulas were reverse-engineered directly from a real GoMining account's miner-detail screenshot (electricity + service cost breakdown) and then cross-checked against ~7 weeks of that account's actual daily income history — the model reproduces real on-platform daily costs to within **~0.3%**.

```
elec_0  = (kWh_rate × 24 × W/TH) / GMT_price / 1000        (GMT/TH/day, 0% discount)
service_0 = service_constant / GMT_price                     (GMT/TH/day, 0% discount)

daily_GMT = TH × (elec_0 + service_0) × (1 − total_discount)
weekly_OPEX_GMT = daily_GMT × 7

locked_GMT ≈ locked_days × TH × (elec_0 + service_0)
liquid_GMT ≈ liquid_days × TH × (elec_0 + service_0)

weekly_dividend_GMT = locked_GMT × APR ÷ 52
required_locked_GMT = (target% × weekly_OPEX_GMT × 52) ÷ APR
breakeven_APR        = (target% × weekly_OPEX_GMT × 52) ÷ current_locked_GMT
```

GoMining's own "days covered" figure (shown on the Maintenance Discount page) is always calculated at the **0% discount rate** — that's why the locked/liquid GMT conversion above doesn't apply your discount stack, even though your real weekly bill does.

## On the APR assumption — read this before trusting any output

Lock dividend APR is the single biggest unknown in the whole calculation, and every source for it has real problems:

- **GoMining's own dashboard** has shown APR in the ~77–92% range across recent mint cycles — self-reported, with no disclosed methodology.
- **A widely-cited "12.6%" figure** traces to a single blog post reporting one person's lock — real numbers (13,594 GOMINING locked, 1,714 GOMINING earned), but the post never states a time period, so it may not even be an annualized rate.
- **The most defensible number available**: GoMining's own in-app Lock Calculator, queried live against a real account, quoted **22.58% APR** for a ~2,000 GOMINING lock at max duration — matching the platform's own "Cycle 162: APR 22.68%" badge shown at the same time. That's a live, first-party, directly-observed figure, not a secondhand claim. It's the calculator's default.
- Locking far more than that (tested up to the calculator's 10,000,000 GOMINING slider max) dilutes the quoted rate slightly — down to ~21.4% — but that effect is only meaningful at whale scale.
- This rate is **not fixed**. It moves cycle to cycle, and it decays for any individual lock over time unless the lock is "re-maxed" (extended back to the full duration) — which is also why the original Flywheel strategy treats weekly re-maxing as a required step, not an optional one.

**Bottom line: treat every "required GMT" and "breakeven APR" result as conditional on the APR you select.** This tool is for modeling scenarios, not predicting returns.

## Not financial advice

This is a calculator, not a recommendation. GoMining is an offshore, unlicensed platform; locked GMT is illiquid for up to 4 years; and every number here is a snapshot that will move. Verify current figures in-app before acting on anything this tool produces.

## Running it locally

It's a single static HTML file with no dependencies — open `index.html` in any browser, or serve the folder with anything that serves static files.
