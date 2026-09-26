# GoMining Flywheel — GMT Lock Calculator

A single-page, no-build calculator for [GoMining](https://gomining.com) miners running the **Flywheel Strategy** (originally written up by u/DracoF on [r/gomining](https://www.reddit.com/r/gomining/)), extended to a "125% OPEX coverage" target: how much GMT do you need locked so the lock dividends alone cover your weekly electricity + service costs?

**[Live site →](https://jdobbsclt.github.io/gomining-flywheel-calculator/)**

## What it does

You enter your hashpower, efficiency, power/service costs, your VIP tier and service streak, and the GMT you hold **locked** and **liquid** (as "days covered" from GoMining's Maintenance Discount page, or as GMT — whichever you edit last is kept and the other is derived). The calculator then works out:

- your **token discount** automatically (see below), and your total maintenance discount stack;
- your weekly OPEX in USD and GMT;
- the **lock APR** GoMining's veGOMINING model actually gives you, for the lock period you pick (1 week up to Max) — the APR is a *result*, not a setting, because it depends on how much you lock and for how long;
- your current dividend and OPEX coverage;
- the **required locked GMT** to hit your coverage target (default 125%), with two plans: lock all your liquid GMT too, or keep it liquid — and how much of the gap your liquid GMT covers and how much you would need to buy;
- the same answer for a 1-, 2-, 3-year and Max lock.

Two of the inputs are not editable because GoMining sets them weekly: the **Mining Mode discount** (`mining-mode.json`) and the **lock reward pool and total votes** (`lock-model.json`). Both are refreshed automatically each night (see below).

## Where the numbers come from

**Maintenance costs.** Reverse-engineered from a real GoMining account's miner-detail screenshot (electricity + service cost breakdown) and cross-checked against ~7 weeks of that account's actual daily income history — accurate to within **~0.3%**:

```
elec_0    = (kWh_rate × 24 × W/TH) / GMT_price / 1000        (GMT/TH/day, 0% discount)
service_0 = service_constant / GMT_price
perDay_0  = TH × (elec_0 + service_0)                        (GMT/day, whole farm)
weekly_OPEX_GMT = 7 × perDay_0 × (1 − total_discount)
```

**Discounts add together**: token + service streak + VIP + mining mode (checked against GoMining's Maintenance Discount page: 20 + 2.7 + 1.8 + 1.35 = 25.85%).

**Token discount** is +1% for every 18 days of maintenance your **locked + liquid** GMT covers (0% below 18 days, capped at 20% from 360 days) — GoMining's documented stepped table. GoMining counts those "days" **after** your non-token discounts (VIP, service, mining mode) and before the token discount:

```
maint_days     = (locked_GMT + liquid_GMT) ÷ (perDay_0 × (1 − VIP − service − mining))
token_discount = min(20%, floor(maint_days ÷ 18) × 1%)
```

(An earlier version of this calculator assumed the days were counted at the 0%-discount rate; a real account's numbers show GoMining's own figure matches the after-non-token-discounts version — 477 days displayed vs 477.9 computed, where the 0% version would give 450.) Because locked and liquid GMT both count, locking your liquid GMT costs you nothing in token discount.

**Lock rewards.** GoMining's veGOMINING lock works like this, and this calculator reproduces GoMining's own in-app Lock Calculator with it (weekly reward within **0.01%** for locks of 1 to 10,000,000 GMT, and the APR at every lock period):

```
votes v       = GMT_locked × (time_left ÷ 4 years)
weekly_reward = POOL × v ÷ (T_others + v)          POOL = weekly reward pool, T_others = everyone else's votes
APR           = (365 ÷ 7) × weekly_reward ÷ GMT_locked
```

More GMT locked means more of the pool but also more dilution of your own share, so the APR falls as a lock grows (about 22.7% for a small max lock, ~22.1% at 5M GMT, ~21.5% at 10M). A lock's votes also **decay weekly** unless you re-extend it — this tool assumes you keep it at the period you choose.

**Required lock.** Holding more GMT raises your token discount, which lowers your costs, which lowers the lock you need. So the calculator tries every token-discount tier (0–20%) and reports the smallest lock that works. This was checked against an independent brute-force search over thousands of random scenarios (identical to within 0.000003%).

## The automatically-updated files

Both files are kept current by a nightly job in [`gomining-servicetap`](https://github.com/jdobbsclt/gomining-servicetap), which already loads GoMining's dashboard every night.

**`mining-mode.json`** — the extra Mining-mode discount, set each week by the veGOMINING vote:

```json
{ "value": 1.35, "changed_at": "…", "checked_at": "…" }
```

**`lock-model.json`** — the weekly reward pool and total votes that drive the lock model:

```json
{ "total_votes": 183317000, "weekly_pool_gmt": 797632, "changed_at": "…", "checked_at": "…" }
```

- The page loads both on every view.
- `checked_at` is when the values were last verified. If that's **more than 7 days old** (measured with the web server's clock, not the visitor's), the page shows an amber warning instead of "verified", because it usually means the automatic update has stopped.
- If a file is missing or malformed, the page falls back to built-in defaults and shows the same amber warning. Values must be real numbers within sane ranges, so a null, string or zero can never silently replace them.
- The pool and total votes move every week. Confirm against GoMining's own Lock Calculator (app.gomining.com → Governance → My lock → Calculate) before relying on any result.

## Not financial advice

This is a calculator, not a recommendation. GoMining is an offshore, unlicensed platform; locked GMT is illiquid for up to 4 years; and every number here is a snapshot that will move. Verify current figures in-app before acting on anything this tool produces.

## Running it locally

It's a single static HTML file with no dependencies — open `index.html` in any browser, or serve the folder with anything that serves static files. (Opened straight from disk, browsers block the page from reading the two JSON files, so it shows built-in default values with the amber warning. Serve the folder, e.g. `python -m http.server`, to see the real values.)
