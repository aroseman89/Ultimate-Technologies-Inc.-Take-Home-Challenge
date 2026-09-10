# Part 2 — Experiment and Metrics Design

**Setup:** Gotham (night-active on weekdays) and Metropolis (day-active on weekdays) have
complementary weekday demand, plus a two-way toll bridge that discourages driver partners from
serving both cities. Ultimate proposes reimbursing tolls to encourage partners to work both
sides.

## 1) Key measure of success

**Primary metric: the share of active driver partners who complete at least one trip in *both*
cities within a given week (a "dual-city" or "bridge-crossing" rate).**

Why this metric over alternatives:

- It directly measures the behavior the intervention targets — *partners serving both cities* —
  rather than a downstream outcome that many other things also affect.
- It's a simple binary-per-driver-per-week measure, easy to compute from trip logs (city of
  pickup) and easy to communicate to the operations team.
- It's harder to game or misread than nearby alternatives:
  - **Total tolls reimbursed** measures spend, not behavior change — a few partners crossing many
    times would inflate it while adoption stayed flat.
  - **Total trips or total revenue** conflates the toll program with organic demand growth,
    seasonality, or pricing changes in either city.
  - **Number of bridge crossings** rewards partners who commute back and forth without
    necessarily *working* meaningful hours on both sides.

A secondary/guardrail metric worth tracking alongside it: **total completed trips across both
cities combined**, to confirm the reimbursement isn't simply reshuffling where existing partners
drive without growing overall supply, and **partner earnings/hour**, to confirm the shift isn't
making partners worse off net of the toll cost being reimbursed vs. their time.

## 2) Experiment design

### a) Implementation

- **Unit of randomization: individual driver partner**, not city or time period. Randomizing at
  the city level would leave only two units (Gotham, Metropolis) — no way to get statistical
  power. Randomizing at the partner level lets every partner serve as their own comparison
  group member alongside everyone else.
- **Design: two-arm randomized controlled trial.**
  - *Treatment*: partners who are active in Gotham or Metropolis are opted into full toll
    reimbursement (both directions) for a fixed trial period (e.g., 6–8 weeks, long enough to
    span multiple weekly cycles and let habits form, short enough to limit exposure if it
    backfires).
  - *Control*: partners continue paying tolls as normal.
- **Stratify randomization** by each partner's pre-period home city and pre-period activity
  level (e.g., trips/week tercile), then randomize within strata. This balances the two arms on
  the factors most likely to confound the outcome — a partner who already crosses occasionally is
  a different case from one who's never crossed.
- **Pre-registration**: define the dual-city metric, the trial window, and the primary
  statistical test *before* looking at results, to avoid p-hacking on whichever cut looks best
  afterward.
- Track the metric **weekly per partner** for a few weeks pre-trial (baseline) and throughout the
  trial, so the analysis can use each partner's own baseline rate as a covariate (see below).

### b) Statistical test

- **Primary analysis: difference-in-differences (DiD)** comparing the change in each partner's
  weekly dual-city rate (trial period vs. their own pre-trial baseline) between treatment and
  control groups. This nets out any partner-level fixed differences and any market-wide trend
  (e.g., organic growth in both cities) that would otherwise confound a simple treatment-vs-control
  comparison.
  - Implement as a linear regression (or logistic regression, since the per-week outcome is
    binary) of `dual_city_indicator` on `treatment`, `post_period`, and their interaction,
    clustering standard errors by partner (since each partner contributes multiple weekly
    observations that aren't independent of each other).
  - The interaction coefficient (`treatment × post`) is the estimated causal effect; test it
    against zero.
- **Secondary checks**: a simple two-proportion z-test (or chi-square) on the raw treatment-vs-control
  dual-city rate during the trial period, as a robustness check that doesn't rely on the DiD
  parallel-trends assumption.
- **Power analysis** before launch: use the pre-trial baseline dual-city rate and expected partner
  count to estimate the minimum detectable effect at conventional power (80%) and significance
  (5%), so the team knows going in whether the trial can realistically detect an effect worth
  caring about, and can extend the trial window or partner pool if not.

### c) Interpreting results & caveats

- If the interaction term is positive and significant, and the guardrail metrics (total trips,
  earnings/hour) haven't degraded, that supports rolling out full reimbursement — with a
  cost/benefit check on whether the increase in dual-city serving is worth the reimbursement
  spend relative to the incremental trips it generates.
- If the effect is positive but small, consider whether the reimbursement amount, not the
  concept, is the constraint (e.g., a partial subsidy might get most of the effect at lower
  cost) — this would need a follow-up dose-response test.
- If there's no effect, consider whether the toll was ever the binding constraint at all — time
  cost of crossing, unfamiliarity with the other city's streets/passengers, or a preference to
  stay local might dominate; those aren't addressed by reimbursement.
- **Caveats to flag to the operations team:**
  - *Novelty effects*: a temporary reimbursement announcement could produce a short-lived spike in
    crossings that fades — the trial window should be long enough to distinguish a durable habit
    change from novelty.
  - *Spillovers/contamination*: control-group partners who work alongside treated partners (same
    dispatch pool, same driver forums/social networks) may hear about the toll reimbursement and
    change behavior anyway, biasing the estimated effect toward zero. Worth asking partners
    directly, or checking for behavior changes in control partners who are geographically/socially
    close to many treated partners.
  - *Generalizability*: results from a 6–8 week trial with reimbursement may not hold if the
    subsidy becomes permanent and partners' response saturates or fades; a longer post-launch
    monitoring period is worth planning regardless of the trial's outcome.
