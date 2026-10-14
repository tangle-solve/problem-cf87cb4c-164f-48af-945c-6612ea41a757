<!-- solve-problem:cf87cb4c-164f-48af-945c-6612ea41a757 -->
# Robust difficulty control with strategic supply

A [Tangle Solve problem](https://solve.tangle.tools/problems/cf87cb4c-164f-48af-945c-6612ea41a757).

## What is the problem?

In proof-of-work blockchains, a difficulty adjustment algorithm (DAA) maps the observed history of block timestamps and heights to a target difficulty for the next block, so that the long-run block arrival rate matches a target (e.g., one block per 10 minutes in Bitcoin). The classical analysis treats miners as non-strategic and hashpower as an exogenous stochastic process. The open problem is to **design (and prove properties of) a DAA that is simultaneously robust to three coupled disturbances**:

1. **Timestamp manipulation**: miners self-report timestamps; a coalition can shift reported times within a skew tolerance (e.g., Bitcoin accepts a timestamp iff it exceeds the median of the previous 11 timestamps and is below network-adjusted time + 2 hours). Manipulated timestamps feed directly into the DAA's estimate of block rate.
2. **Hashpower shocks**: abrupt changes in total hashpower (hardware deployment, outage, coin-switching by profit-switching pools) cause transient block-rate deviations; the DAA must recover fast.
3. **Strategic supply**: total hashpower is chosen endogenously by profit-maximizing miners responding to expected reward per unit hash — so the DAA itself feeds back into supply. A DAA that adjusts aggressively can induce oscillatory hashpower dynamics (or be exploited to farm high-difficulty/high-revenue epochs).

Formal setting: state variables are difficulty $D_t$, true hashpower $H_t$, reported timestamps $\{\hat{t}_i\}$, and true times $\{t_i\}$ with $\hat{t}_i \in [t_i - \delta, t_i + \delta']$ per protocol rules. Blocks arrive as a Poisson process with rate $H_t/D_t$. Miners choose $H_t$ to maximize expected revenue minus cost. Goal: design the DAA update rule $\hat{D}_{t+1} = f(\text{history})$ minimizing recovery time to target block rate under admissible timestamp manipulations, and proving no profitable manipulation/deviation strategy exists in the hashpower supply game induced by $f$. The precise robustness objective (worst-case regret? $H_\infty$/minimax bound? dominant-strategy incentive compatibility?) is not fixed by the census and is part of the problem — a solver should pick and justify one.

## Why does it matter?

DAAs are consensus-critical, live infrastructure: Bitcoin's DAA was emergency-hard-forked in 2017 precisely because slow recovery under hashpower exodus threatened liveness, and smaller chains (BCH, ETC, BTG) have suffered deep reorgs enabled by oscillating hashpower driven by their DAAs. A DAA provably robust to timestamp manipulation closes known (unfixed-in-practice) attack surfaces on chain-quality and fee/reward estimation, while robustness to strategic supply matters for any coin using ASIC-resistant or rented hashpower. Downstream, such a design would yield the first consensus-layer clock that is simultaneously a mechanism — a result of interest to robust control, algorithmic game theory, and protocol engineering.

## What's already known?

- **Kraft (2016)** gives the classical control-theoretic analysis: a DAA is a feedback controller stabilizing block rate; he proves stability (convergence to target rate) under i.i.d. or Markovian hashpower and honest timestamps, essentially treating the DAA as a PI controller on the timespan/height ratio. No adversarial or strategic analysis.
- Bitcoin's DAA and its 2017 replacement (aserti3-2d on BCH) and Digishield/DarkGravityWave on altcoins are analyzed empirically; known pathologies include slow recovery (pre-2017 Bitcoin) and hashpower oscillation from profit switching (BCH post-2017, documented empirically by communities; no rigorous theory — verify).
- **Timestamp manipulation**: known folklore results that bounded timestamp shifts induce bounded difficulty error, and attacks exploiting the 2-hour rule have been discussed informally; rigorous bounds appear only in scattered/secondary sources (verify). Median-based acceptance (MTP) limits the leverage of small coalitions but is not analyzed against strategic coalitions jointly with supply.
- **Noda, Okumura & Hashimoto (2026)** build an economic model where hashpower supply responds to expected reward and the DAA closes the loop; this is the "supply-response model" referenced by the census. It characterizes when the coupled system is stable/oscillatory, but (per census) treats timestamps as honest and gives analysis, not a design that is robust by construction. (Verify details — the census is the source for this paper's content.)
- **Gap**: no paper combines (i) minimax/adversarial timestamp perturbations, (ii) fast recovery under shocks (the fast-recovery vs. noise-amplification tradeoff — aggressive gains amplify manipulation), and (iii) strategic hashpower agents. The census explicitly places the target "beyond classical control analysis and the 2026 supply-response model."

## What would count as solving it?

Any of the following, in increasing strength:

1. **Upper bound / construction**: an explicit DAA $f$ with a proof that (a) for all timestamp strategies of coalitions controlling up to fraction $\alpha$ of hashpower within protocol skew bounds, block-rate deviation over any window $T$ is at most $\varepsilon(\alpha, T)$; and (b) under the supply-response model of Noda et al., the induced dynamical system is asymptotically stable with stated convergence rate.
2. **Lower bound / impossibility**: a proof that no DAA satisfies both fast recovery (given recovery-time target) and manipulation resistance (given deviation bound) — i.e., a formal tradeoff theorem quantifying the "fast recovery vs. manipulation resistance interact" claim.
3. **Matching bounds**: matching upper and lower bounds pinning the Pareto frontier of (recovery time, manipulation-induced rate deviation, supply-induced oscillation amplitude).
4. **Mechanism-design variant**: a DAA (possibly with timestamp-vote rules or reward timing changes) under which honest reporting plus non-strategic supply is an approximate equilibrium of the joint game.

A referee wants: a precise adversary model (who controls timestamps, what skew is allowed, what hashpower fraction), a precise dynamical model of supply, and either a proven guarantee or a proven impossibility. Simulations alone do not solve it (though they are useful evidence).

## How to start

1. **Read Kraft (2016) first** — Section on the linearized model. Recast the DAA as a linear feedback loop: block-interval estimate → controller gain → difficulty. This gives you the plant model and the classical stability conditions; everything after is robustifying this loop.
2. **Read Noda–Okumura–Hashimoto (2026)** to import the supply-response dynamics: hash supply as a function of expected reward per hash, which depends on difficulty, closing the loop. Write the coupled two-timescale system (fast: block arrivals; slow: supply adjustment) and check stability of your candidate DAA in it.
3. **Model the timestamp adversary**: replace observed intervals by adversarially shifted ones under MTP-style acceptance rules; compute the induced multiplicative error in the block-rate estimate as a function of coalition size and skew bound. This is small-disturbance analysis on the linearized loop; small-gain arguments are the natural tool for the joint robustness statement.
4. **Main obstacle**: robustness to manipulation wants small controller gain; fast recovery wants large gain; strategic supply adds an endogenous gain. Expect to state and prove a tradeoff (item 2 above) before attempting an optimal design — this is likely the most tractable first publishable result.

## Key references

- Daniel Kraft. *Difficulty control for blockchain-based consensus systems*. Computing and Informatics 33(2), 2016 (also appeared 2015 online). DOI: 10.1007/s12083-015-0347-x — https://doi.org/10.1007/s12083-015-0347-x. Models a general DAA as a controller stabilizing block-arrival rate and proves stability for a class of algorithms under non-adversarial hashpower dynamics; the baseline "classical control analysis."
- Kenta Noda, Shigeyuki Okumura, Yoshinori Hashimoto. *An Economic Analysis of Difficulty Adjustment Algorithms in Proof-of-Work Blockchain Systems*. International Economic Review, 2026. DOI: 10.1111/iere.70028 — https://doi.org/10.1111/iere.70028. Models miners' hashpower supply as endogenous to the DAA and analyzes stability/oscillation of the coupled system; the "2026 supply-response model" this problem must go beyond. (Content summary is from the census record; verify against the paper.)
- (Supplementary, verify) Literature on Bitcoin's 2017 DAA change and aserti3-2d, e.g., the BCH aserti3-2d specification and "difficulty oscillation" analyses by Fatehi/stone.cn — empirical evidence for supply-driven oscillation, useful for model calibration.

## Warnings

- **Ambiguity in the objective**: the census does not fix the robustness metric (worst-case deviation, regret, $H_\infty$ norm, equilibrium concept). Pick one explicitly and state it; a vague "robust" claim will not referee.
- **Adversary model subtlety**: timestamp manipulation is bounded by protocol rules (median-past, future-limit), so the adversary is *constrained*, not arbitrary. Use the constraints or your bounds will be vacuous/too pessimistic.
- **Common mistake**: analyzing recovery under exogenous hashpower shocks while ignoring that the shock itself responds to the DAA (profit-switching pools). The loop must be closed through supply, not treated as an open-loop disturbance.
- **Dead end to avoid**: purely stochastic (non-strategic, non-adversarial) stability proofs — this is exactly what Kraft (2016) already did; the novelty is adversarial + strategic.
- The Noda et al. (2026) paper's exact scope is known only from the census record; read it before assuming what it does or doesn't cover.

---
*`BCGT-000242` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Upper bound / construction**: an explicit DAA $f$ with a proof that (a) for all timestamp strategies of coalitions controlling up to fraction $\alpha$ of hashpower within protocol skew bounds, block-rate deviation over any window $T$ is at most $\varepsilon(\alpha, T)$; and (b) under the supply-response model of Noda et al., the induced dynamical system is asymptotically stable with stated convergence rate.
2. **Lower bound / impossibility**: a proof that no DAA satisfies both fast recovery (given recovery-time target) and manipulation resistance (given deviation bound) — i.e., a formal tradeoff theorem quantifying the "fast recovery vs. manipulation resistance interact" claim.
3. **Matching bounds**: matching upper and lower bounds pinning the Pareto frontier of (recovery time, manipulation-induced rate deviation, supply-induced oscillation amplitude).
4. **Mechanism-design variant**: a DAA (possibly with timestamp-vote rules or reward timing changes) under which honest reporting plus non-strategic supply is an approximate equilibrium of the joint game.

A referee wants: a precise adversary model (who controls timestamps, what skew is allowed, what hashpower fraction), a precise dynamical model of supply, and either a proven guarantee or a proven impossibility. Simulations alone do not solve it (though they are useful evidence).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/cf87cb4c-164f-48af-945c-6612ea41a757?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000242.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000242)
