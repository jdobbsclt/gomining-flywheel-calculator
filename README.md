# GoMining Flywheel Calculators

Single-page, no-build calculators for [GoMining](https://gomining.com) miners planning a **Flywheel**, in three tabs that share the same farm and GMT inputs:

- **Traditional GMT Flywheel** — the rules-based strategy by u/DracoF ([write-up](https://www.reddit.com/r/gomining/comments/1sbqys5/the_flywheel_strategy/)): a status headline and this week's action, a weekly checklist, and the GMT and dollars needed to build 360 locked / 140 unlocked days (described below).
- **Modified Flywheel** — the variation by u/InterestingEngine146, "Audacity" ([write-up](https://www.reddit.com/r/GoMiningDiscussion/comments/1uxjlnh/a_modified_flywheel/)): 400 locked / 100 unlocked days, buy TH on the marketplace with GMT instead of reinvesting directly, and a BTC cutoff that switches between accumulating and profit-taking (described below).
- **OPEX 125% Flywheel** — how much GMT do you need locked so the lock dividends alone cover your weekly electricity + service costs? (The original calculator; described below.)

The page opens on **Traditional**. The open tab is kept in the URL (`#traditional`, `#modified`, `#opex`), so a tab can be linked to.

**Colors in every Results panel:** a GMT category you meet or exceed is **green**; one that is under its required amount is **red**. Anything that is only information stays neutral.

**[Live site →](https://jdobbsclt.github.io/gomining-flywheel-calculator/)**

## The shared inputs (above the tabs)

Farm & market (TH, efficiency, GMT price, power rate, service constant), the maintenance discount stack, and your GMT position (as days or GMT; locked and liquid). Every tab reads the same numbers, so you enter them once.

## What the OPEX 125% tab does

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

## The Traditional GMT Flywheel tab

Implements DracoF's rules: lock 360 days of GMT at the maximum period, keep 140 days liquid (500 in total); reinvest in GMT while building and below 500 total days, and in TH or BTC above; optional triggers to buy miners above 640 days and sell extra miners below 360. It reads your shared farm and GMT inputs, and its **Results** use the same format as the OPEX tab (a headline answer, four stat cards, a progress bar and a table):

- **Headline:** which stage you are in (building the locked side, filling the unlocked buffer, built, or above the trading trigger) and what to do about it, with your total days against the 500-day target.
- **Stat cards:** locked days and unlocked days against their targets, the **GMT to acquire** to satisfy both (unlocked GMT can be locked but locked GMT cannot be unlocked, so this is the fewest extra days of GMT that meets both targets), and the **price cushion**: how far the GMT price can fall before your total days drop under 360 and you lose the 20% token discount. Maintenance is priced in dollars, so your days shrink in step with the GMT price.
- **Table:** GMT and dollars for each target, what you hold, and the gap, plus GoMining's 2.25% reinvest fee if you build by reinvesting BTC earnings into GMT.
- **Weekly checklist:** re-max the lock, check your days, top up the lock, set the reinvest strategy, and the optional buy/sell triggers.
- The targets (360 / 140 / 14 unlocked while building) are editable.

## The Modified Flywheel tab

Implements Audacity's variation, in the same Results format (headline, four stat cards, progress bar, tables):

- **Targets:** 400 locked / 100 unlocked days by default, with an unlocked floor (default 50) below which the buffer is too thin. All editable.
- **Phase headline:** compares the BTC price with your **BTC cutoff** (default $100,000, your own assumption). Below it: *accumulation*, keep reinvesting into GMT and TH. At or above it: *profit-taking*. The BTC price loads live from CoinGecko and can be overridden.
- **Buying TH:** compares reinvesting directly at GoMining's ask (with the VIP bonus: +5% from Silver I, +10% from Diamond I) with buying listed TH on the marketplace using GMT, at a discount below the ask and after GoMining's 2.25% reinvest fee:

  ```
  direct      TH per $ = (1 + VIP bonus) ÷ ask
  marketplace TH per $ = (1 − 0.0225) ÷ (ask × (1 − marketplace discount))
  ```

  At a 20% discount and a 5% VIP bonus this gives about 16.4% more TH, which reproduces Audacity's "about 15% more" once the fee is included. The ask price is estimated from efficiency ($12.53 at 15 W/TH plus $2.67 per W/TH of upgrade, from Audacity's July 2026 numbers, about $20.54 at 12 W/TH) and can be overwritten.
- **Greedy Machines:** the years of free weekly TH growth needed to pay back a premium over the market price: `ln(1 + premium) ÷ (52 × ln(1 + weekly growth))`. At 0.16% a week, a 20% premium needs about 2.2 years, 30% about 3.2 and 50% about 4.9 (the extra TH also needs more maintenance, so the real break-even is longer).

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
