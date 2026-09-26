# GoMining Flywheel — GMT Lock Calculator

A single-page, no-build calculator for [GoMining](https://gomining.com) miners running the **Flywheel Strategy** (originally written up by u/DracoF on [r/gomining](https://www.reddit.com/r/gomining/)), extended to a "125% OPEX coverage" target: how much GMT do you need locked so the dividend yield alone covers your weekly electricity + service costs?

**[Live site →](https://jdobbsclt.github.io/gomining-flywheel-calculator/)**

## What it does

Almost every input is editable — total hashpower, efficiency (W/TH), GMT price, power rate, and three of the four independent maintenance-discount components (token discount slider, service-streak dropdown, VIP-tier dropdown with all 21 tier names from Bronze I to Elite) — and results update live. The fourth, **Mining Mode**, is set by GoMining's weekly veGOMINING vote for everyone in Mining mode, so it isn't editable: the page reads it from `mining-mode.json` (see below).

- Weekly OPEX in USD and GMT
- Locked and liquid GMT — enter either your "days covered" figures (from GoMining's Maintenance Discount page) or your GMT amounts directly; whichever you edit last is kept and the other is derived from it
- Weekly dividend at an assumed lock APR
- **Required locked GMT** to hit a chosen OPEX-coverage target (default 125%)
- **Breakeven APR** — the yield at which your current locked GMT already meets the target
- A scenario row at GoMining's own live 22.6% quote (~2K GOMINING lock): required locked GMT, gap vs. your current lock, and extra capital needed

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

## The Mining Mode discount (`mining-mode.json`)

GoMining sets the extra Mining-mode maintenance discount each week from the veGOMINING vote and Burn & Mint cycle, so no fixed number is right for long. This repo keeps the latest value in `mining-mode.json`:

```json
{ "value": 1.35, "changed_at": "…", "checked_at": "…" }
```

- The calculator loads it on every page view. `value` is a percent.
- `checked_at` is when the value was last verified against the app. If that's **more than 7 days old** (measured with the web server's clock, not the visitor's), the page shows an amber warning instead of "verified", because it usually means the automatic update has stopped.
- If the file is missing or malformed, the page falls back to a built-in default and shows the same amber warning. It never silently treats a bad value as 0%.
- Confirm the number against the Maintenance Discount page in the GoMining app before relying on any result.

## Not financial advice

This is a calculator, not a recommendation. GoMining is an offshore, unlicensed platform; locked GMT is illiquid for up to 4 years; and every number here is a snapshot that will move. Verify current figures in-app before acting on anything this tool produces.

## Running it locally

It's a single static HTML file with no dependencies — open `index.html` in any browser, or serve the folder with anything that serves static files. (Opened straight from disk, browsers block the page from reading `mining-mode.json`, so it shows the built-in default Mining Mode value with the amber warning. Serve the folder, e.g. `python -m http.server`, to see the real value.)
