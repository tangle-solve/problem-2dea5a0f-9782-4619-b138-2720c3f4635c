<!-- solve-problem:2dea5a0f-9782-4619-b138-2720c3f4635c -->
# Stablecoin-bank contagion with settlement calendars

A [Tangle Solve problem](https://solve.tangle.tools/problems/2dea5a0f-9782-4619-b138-2720c3f4635c).

## What is the problem?

The problem is to build and **validate** (empirically and experimentally) a contagion model of a stablecoin system in which the following interact:

- **Continuous trading:** the stablecoin trades on secondary markets at a floating price, so runs are triggered by price deviations from peg rather than a single discrete withdrawal decision.
- **Bank closures / settlement calendars:** the fiat rails backing redemptions (custodial bank accounts, payment systems) are only available at discrete times (weekends, holidays, intraday cut-offs), so an investor who wants out at time *t* may wait until time *t + Δ* for settlement.
- **Shared custodians:** multiple stablecoin issuers (or banks backing them) use the same custodian or reserve bank, creating correlated exposure to a single intermediary's failure or freeze.
- **Redemption queues:** redemptions are processed FIFO (or with some priority rule) subject to limited reserve liquidity, so a run manifests as a growing queue, and queue position determines loss given default.

Formal setting: a set of issuers *i = 1..n* with reserves *R_i(t)* held at custodians *c ∈ C*; a demand/redemption arrival process (e.g., a Hawkes or Poisson process whose intensity depends on the market price *p(t)* of the stablecoin); a service mechanism that is only "on" during open settlement windows *W ⊂ [0,T]*; and a default/insolvency rule (e.g., *R_i(t) − Q_i(t) < 0* where *Q_i(t)* is queued redemption demand). The question is whether a canonical model of this type, calibrated to 2024–2026 stablecoin data, reproduces observed contagion dynamics — and specifically whether it can **separate operational delay (gridlock) from true insolvency** as sources of de-pegging and loss. The census record is informal; the precise formulation above is the most reasonable reading, but the exact arrival process, priority rule, and calibration target are left to the solver.

## Why does it matters?

If regulators cannot distinguish "the issuer is solvent but the banks are closed" from "the issuer is insolvent," they will either bail out solvent-but-gridlocked issuers or let insolvent ones pay out slowly and fail. A validated model directly informs the design of reserve transparency rules, custodian diversification requirements, and emergency liquidity facilities for stablecoins — live policy questions at the Federal Reserve and BIS after the 2023 USDC/Silicon Valley Bank de-peg, which was a pure operational-delay event misread by many as insolvency. It also connects to systemic-risk assessment: shared custodians are a contagion channel that current single-issuer run models (Diamond–Dybvig-style) miss entirely.

## What's already known?

- **Classical bank-run theory** (Diamond–Dybvig) treats runs as coordination failures with instantaneous withdrawal; it has no settlement calendar and no secondary market price.
- **Stablecoin run models** (e.g., work following the 2022 Terra/UST collapse) typically model a single issuer with perfect, continuous redemption against full reserves, showing first-mover advantage under partial backing. These models do not handle queues, custodians, or closure windows. (verify — the exact frontier for 2026 multi-issuer models should be checked.)
- **Payment-system literature** on gridlock and liquidity-saving mechanisms studies settlement queues in RTGS systems, but not market-price-driven runs.
- **Queueing theory with server vacations** gives the mathematical machinery for "service unavailable on weekends," but standard results assume exogenous arrival rates, not endogenous run dynamics.
- **Carapella, Lubis and Vardoulakis (2026)** provides the motivating empirical picture of intermediation and integration risks (concentration at shared custodians, bank integration), but is analysis, not a contagion model.

No published model combines all four features with calibration; that is the gap.

## What would count as solving it?

This is an **experimental/validation** problem. Any of the following would count:

1. **A calibrated agent-based or stochastic model** incorporating (a)–(d) above, fitted to at least one historical de-peg episode (USDC March 2023 is the canonical operational-delay case), that reproduces the observed price path and queue/redemption volumes within stated error bounds.
2. **A separation theorem/diagnostic:** a statistic computable from observable data (market price, redemption queue length, time-to-next-settlement-window) that provably or empirically distinguishes "gridlock de-peg" from "insolvency de-peg," validated on synthetic data where ground truth is known.
3. **Comparative statics on custodian structure:** demonstration (simulation + analytic approximation) of how contagion extent scales with the number of shared custodians — e.g., showing a threshold concentration level beyond which one custodian's closure triggers multi-issuer runs.
4. **Robustness:** evidence the conclusions survive misspecification of the arrival process.

A referee would want: open code/data, calibration to real episodes, and a falsifiable separation criterion.

## How to start

1. Read Carapella, Lubis & Vardoulakis (2026) first — it defines the institutional facts (which custodians, what integration) your model must capture, and is the source that "sharpens" this problem.
2. Build the minimal single-issuer model: price *p(t)* determined by impatient traders, redemptions queued at rate λ(p) with intensity spiking when *p < 1 − ε*, service only in windows *W*. Analyze the deterministic fluid limit — when does the queue drain after the window reopens? This already yields the delay/insolvency dichotomy cleanly.
3. Add a second issuer sharing the custodian; model correlated exposure as simultaneous vacation for both. Use the server-vacation queueing literature (e.g., queueing with vacations) for single-node results, then simulate.
4. Calibrate to USDC/SVB March 2023 (public price data on CoinGecko; Circle's disclosures on redemption delays) and test the separation statistic. Main obstacle: redemption-queue data is mostly proprietary — you may need to proxy queues from on-chain burn rates and secondary-market discounts.

## Key references

- Carapella, Francesca, Titania Lubis, and Andreas Vardoulakis (2026). *Stablecoins in 2025: Developments and Financial Stability Implications.* FEDS Notes. DOI: 10.17016/2380-7172.4020. — Documents the intermediation structure (custodian concentration, bank integration) of major stablecoins and frames the financial-stability question this problem targets.
- Diamond, D. and Dybvig, P. (1983). "Bank Runs, Deposit Insurance, and Liquidity." *JPE* 91(3). — Canonical run model; the baseline your setup generalizes (no calendars, no secondary price, full service).
- Klages-Mundt, A. and Minca, A. (2021). "While Stability Lasts: A Stochastic Model of Stablecoins." — arXiv/Netw Econ (verify). — Models stablecoin runs with endogenous withdrawal incentives; closest single-issuer run framework to extend.
- Literature on queueing networks with server vacations (survey-level; any standard reference, e.g., Takagi (1991), *Queueing Analysis*, vol. 1) (verify) — supplies the mathematical tools for closure windows.
- Payment-gridlock / RTGS liquidity literature (e.g., work by Galbiati and Soramäki on settlement systems) (verify) — gridlock mechanics without price-driven runs.

## Warnings

- The census record is a one-line informal statement; the "problem" has no canonical published formulation, so define yours explicitly and defend the choices (arrival process, priority rule, default condition) — reviewers will probe these.
- Do not conflate delay and insolvency in your metric: a model that predicts de-pegs but cannot attribute them is not a solution.
- USDC/March 2023 is your cleanest ground truth (operational delay, issuer solvent); avoid calibrating only on Terra/UST (an algorithmic collapse, different mechanism).
- Custodian identity is often not disclosed precisely; treat concentration parameters as uncertain and do sensitivity analysis.
- Queue data is proprietary; proxies from on-chain burns miss off-chain institutional redemptions — state this limitation.

---
*`BCGT-000333` · flagged for review*

## Acceptance guidance

This is an **experimental/validation** problem. Any of the following would count:

1. **A calibrated agent-based or stochastic model** incorporating (a)–(d) above, fitted to at least one historical de-peg episode (USDC March 2023 is the canonical operational-delay case), that reproduces the observed price path and queue/redemption volumes within stated error bounds.
2. **A separation theorem/diagnostic:** a statistic computable from observable data (market price, redemption queue length, time-to-next-settlement-window) that provably or empirically distinguishes "gridlock de-peg" from "insolvency de-peg," validated on synthetic data where ground truth is known.
3. **Comparative statics on custodian structure:** demonstration (simulation + analytic approximation) of how contagion extent scales with the number of shared custodians — e.g., showing a threshold concentration level beyond which one custodian's closure triggers multi-issuer runs.
4. **Robustness:** evidence the conclusions survive misspecification of the arrival process.

A referee would want: open code/data, calibration to real episodes, and a falsifiable separation criterion.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/2dea5a0f-9782-4619-b138-2720c3f4635c?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000333.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000333)
