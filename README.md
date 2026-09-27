# FMShape

**Conditional flow matching for target-shape reconstruction from high-resolution range profiles in spaceborne integrated sensing and communication — built on a certificate-carrying, falsifiable research platform.**

FMShape reconstructs the 3D point-cloud shape of a ground region of interest (ROI) from its measured wideband high-resolution range profile (HRRP) inside a spaceborne ISAC pipeline, in a *single* sampling step: an optimal-transport conditional flow-matching (OT-CFM) model trained in the latent space of a variational autoencoder, conditioned on the measured profile. At equal compute it matches a 100-step diffusion baseline's reconstruction quality at 50–100× fewer network evaluations, and a theoretical analysis shows the one-step output is exactly the posterior conditional mean — which reframes the collapse of converged one-step maps as the Bayes-optimal behavior rather than a defect.

- **Manuscript**: [`docs/paper/FMShape_draft_v0.1.md`](docs/paper/FMShape_draft_v0.1.md) (v0.2 draft, seven sections, Figs. 1–3, Tables I–IV, 19 references; target venue IEEE TAES)
- **Platform report**: [`TECH_REPORT.md`](TECH_REPORT.md) (full system, every headline number certified / bracketed / falsified in public)
- **Verification discipline**: [`docs/protocol_contamination_audit.md`](docs/protocol_contamination_audit.md) (per-table batch provenance; the paired-protocol standard)

---

## The platform

One reproducible testbed, six layers — everything below is fixed-seed and one-command reproducible:

| Layer | What is here |
|---|---|
| **Orbit & scenario** | Real SGP4 propagation from public TLEs (ISS / Starlink), bistatic satellite→ROI→UE geometry, 30 GHz carrier, 1 GHz sensing bandwidth |
| **RIS** | Per-frame closed-form full-model phase alignment, coordinate ascent with SDR/Lagrangian dual certificates, K=1/2/4/8 reconfiguration-rate trade-off, the structural power-gate identity |
| **Radar sensing** | HRRP formation (centroid-aligned wideband delay projection), ISAR slow-time Doppler profiles, MUSIC / 2D-CFAR classical baselines, CRB information floors |
| **Generative sensing (FMShape)** | PointVAE whitened latent + OT-CFM + 1-D DiT cross-attention + LSTM condition encoder + classifier-free guidance; NFE=1 operating point; one-step distillation frontier |
| **Waveforms** | OTFS / AFDM against the real LEO Doppler (±611 kHz, derived from SGP4), exact OFDM ICI identity |
| **Methodology** | Pre-registered propositions with recorded verdicts, fixed seeds, evidence JSONs, the paired-protocol discipline (see below) |

## Quickstart

```bash
make setup        # environment
make demo         # auto-train + sensing–communication closed loop
make verify       # full verification suite (each headline number: certified, bracketed, or falsified)
```

Key verification targets: `verify-fm-bounds` (ODE order / straightness / crossover), `verify-gen-metrics` / `verify-gen-metrics-n32` (distributional fidelity), `verify-fm-train-scale-paired` / `verify-m3-paired` (paired scaling sweeps), `verify-headline-multiseed` (3-seed paired A/B), `verify-fm-shape` (closed-loop deployment).

## Certificate & falsification ledger

The distinguishing feature of this project is not any single result but the **claim–evidence contract**: every headline number is either certified, bracketed, or publicly falsified — including our own predictions. A selection:

| Claim | Status | Evidence |
|---|---|---|
| RIS coordinate ascent reaches 88.05% of the certified global optimum; design-factor interval [0.732, 0.832] | **Certified** (SDR + Lagrangian dual bracket) | TECH_REPORT §6.7 |
| Single-scatterer path-length CRB 0.5165 mm; MC/CRB = 0.984 | **Certified** (CRB + Monte-Carlo) | TECH_REPORT §6.8 |
| Far-field angle wall: mono-static cross-range localization shortfall 1889× | **Certified** (geometry + Van Trees) | TECH_REPORT §6.4–6.5 |
| ±611 kHz overpass Doppler | **Derived, not measured** (SGP4 peak \|v_r\| ≈ 6.10 km/s, f_d = v_r/λ at λ = 1 cm) | TECH_REPORT §2.1 |
| OFDM ICI identity 28.350% at the real Doppler | **Certified** (closed form = Monte-Carlo 10⁶) | TECH_REPORT §6.8 |
| FM (NFE=1) vs DDPM (NFE=100) | **Sampling-efficiency parity, not superiority** — 3-seed paired differences +4.5% / −20.5% / −4.4%, interval crosses zero | `headline_multiseed.json` |
| Conditioning channel Δ(0) | **Replicated on 3 seeds**: 0.302 / 0.190 / 0.215 (threshold 0.05) | `w1_multiseed.json` |
| ISAR condition augmentation | **2 of 3 seeds**: +6.9% / +3.9% / −99.2% (seed-44 channel functionally inert — reported, not averaged) | `w1_multiseed.json`, paper §VI-D |
| One-step distillation student beats the teacher's mean | **Falsified** at matched budget (both pre-registered propositions fail — the structural price of dispersion) | paper §V-D, §VI-E |
| Training scale closes the oracle gap | **Falsified** on both seeds (paired protocol: no measurable effect; consistent degradation at 4× on one seed) | `fm_train_scale_paired.json` |
| M3 capacity probe (4× generative epochs, narrowband) | **Superseded by paired re-evaluation**: NFE=1 improves consistently (+11.8% / +18.7%) but the gap stays ≈6×; the original cross-run falsification was batch noise | `m3_paired.json` |
| Euler convergence order | **−0.89** on the 32-cloud batch (−0.87 / −0.94 on 16 / 8 clouds — order robust, point estimate batch-sensitive) | `verify_fm_bounds.json` |
| Per-cell evaluation batches are comparable across runs | **Falsified — and fixed**: on-access dataset sampling makes post-training batch draws run-specific; all cross-run comparisons were re-done on paired common batches | `docs/protocol_contamination_audit.md` |

## Repository layout

```
docs/paper/          FMShape manuscript, figures, internal-free submission material excluded
source_code/isac_sat/    experiment suite (train / eval / verify scripts, evidence JSONs)
source_code/isac_sim/    platform library (scenario, channels, waveforms, RIS)
TECH_REPORT.md       full platform report with per-claim certificates
docs/                protocol audit, roadmap, physics audit table
tests/ archive/ assets/    CI, lineage of the original ISAC-Diffu-ISAC docs, figures
```

## Lineage

This repository originated as a fork of the research line of [`ConradLu2740/IRS-Diffu-ISAC`](https://github.com/ConradLu2740/IRS-Diffu-ISAC) and has since diverged substantially (flow-matching generative sensing, paired verification protocols, multi-seed closure). The legacy repository remains available; the full development history of this line is kept locally and available from the authors on request.

## Status

Manuscript in internal review (Zhejiang University of Technology, School of Information Science & Engineering). Simulation-based throughout; measured-data / EM-simulation transfer validation is the identified first item of future work and is stated as such in the manuscript.
