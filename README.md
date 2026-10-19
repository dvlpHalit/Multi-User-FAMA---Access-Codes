# Multi-User FAMA under Heterogeneous Interference: reproduction code

MATLAB/Octave code for

> M. H. Aldemir, M. A. Karabulut, A. F. M. S. Shah and M. Madahana, "Outage Analysis of Multi-User FAMA under Heterogeneous Interference over Block-Correlated Channels with Nakagami-*m* Textures," *IEEE Access*, revised manuscript under review, 2026.

The paper studies a fluid antenna terminal that selects one of *N* strongly correlated ports while *U* − 1 co-channel emitters of unequal power, fading severity and in-aperture profile transmit, including during channel training. It derives the interference-limited outage floor in closed form, its diversity order, the effect of finite training and a parameter-free texture-aware correction for the coupling between blocks, and it introduces a selector that senses the interference in the pilot residual. Every analytical result is checked against a simulation of the fully correlated aperture.

## Contents

The code is in the archive `MAccess.rar` (RAR5; opens with 7-Zip, WinRAR, The Unarchiver or `unar`). It unpacks to

```
Access/        all .m files: configuration, analytical routines, estimators,
               figure and table scripts, revision-step drivers
Access/data/   saved results (MATLAB v7 .mat) from which plot_fig2 ... plot_fig5 redraw Figs. 2-5
```

`fama_config.m` is the single source of truth for every parameter (Table 3 of the paper): *N* = 100 ports over *W* = 3λ, *B* = 4 blocks of *L_b* = 25 ports, block severities *m_b* = {3.0, 2.4, 1.8, 1.4}, emitter severity *m_I* = 2.5, per-emitter SIR 10 dB, threshold 0 dB and *T_p* = 16 pilot symbols.

## Requirements

MATLAB R2021a or later, or GNU Octave 8.4 or later. No toolboxes: Gamma variates, Tricomi's function in (18) and the Gaussian orthant probabilities are implemented in base MATLAB (`fama_gamrnd.m`, `fama_tricomi.m`, `fama_mvnorthant.m`).

## Quick start

```matlab
cd Access
fama_selftest                      % numerical self-test; must end with "=== n passed, 0 failed ==="

cd data                            % redraw Figs. 2-5 from the saved results, no simulation
addpath('..')
plot_fig2; plot_fig3; plot_fig4; plot_fig5   % writes fig2_outage_snr ... fig5_diversity (.eps and .pdf)
```

## Full reproduction

Every revision step has its own driver script, which prints every number that the step contributes to the paper. Run them from `Access/`; the `.mat` files they write land there, and `plot_fig2` ... `plot_fig5` then redraw the figures from them.

```matlab
cd Access
fama_selftest
fig2_sircdf                        % Fig. 1 (writes fig1_sircdf.eps)
run_kume_a                         % ~30 min in Octave
run_kume_b                         % ~15 min
run_kume_c                         % ~20 min; writes fig5_data.mat, needed by run_kume_e
run_kume_d
clear part; run_kume_e             % ~7 h in Octave; data of Figs. 2-5, Tables 4, 7 and 8
clear part; run_kume_f             % Table 6
clear part; run_kume_g
clear part; run_kume_h             % ~2 h in Octave; Tables 2 and 5
plot_fig2; plot_fig3; plot_fig4; plot_fig5
```

MATLAB is considerably faster than Octave. The drivers `run_kume_e` to `run_kume_h` share the variable `part`; `clear part` runs every part. A single part can be run on its own, for example

```matlab
part = 'wstail'; run_kume_e        % Appendix B: block-model columns of Table 8
part = 'tab2';   run_kume_h        % Table 2
```

`run_all.m`, `fig1_sircdf.m`, `fig3_outage_snr.m`, `fig4_floor_vs_U.m`, `fig6_hardening.m` and `diversity.m` belong to the first submission and use its figure numbering; they are kept for traceability.

## What reproduces what

| Object in the paper | Script | Data file written |
|---|---|---|
| Fig. 1, block SIR distribution, (13) | `fig2_sircdf.m` | — |
| Fig. 2, outage against pilot SNR, sensing rule | `run_kume_e`, part `fig2`, then `plot_fig2` | `kume_e_fig2.mat` |
| Fig. 3, floor against *U* and the FGM ceiling | `run_kume_e`, parts `cmp` and `bigU`, then `plot_fig3` | `kume_e_cmp.mat`, `kume_e_bigU.mat` |
| Fig. 4, threshold sweep and effective source count | `run_kume_e`, part `fig4`, then `plot_fig4` | `kume_e_fig4.mat` |
| Fig. 5, diversity on the correlated field | `fig5_diversity.m` and `run_kume_e`, part `fig5`, then `plot_fig5` | `fig5_data.mat`, `kume_e_fig5.mat` |
| Table 2, four selectors | `run_kume_h`, part `tab2` | `kume_h_tab2.mat` |
| Table 3, configuration | `fama_config.m` | — |
| Table 4, anatomy of the residual amplification | `run_kume_e`, part `cmp` | `kume_e_cmp.mat` |
| Table 5, training-limited floors | `table5_training.m` (estimation error, predicted ratio) and `run_kume_h`, part `tab5` (floors, sensing rule) | `table5_data.mat`, `kume_h_tab5.mat` |
| Table 6, heterogeneous emitters | `run_kume_f`, part `table`; `kume_f_tex.m` prints the rows | `kume_f_data.mat` |
| Table 7, comparison with existing analyses | `run_kume_e`, parts `cmp`, `sev`, `corr`, `transfer`, `oracle`, `summary` | `kume_e_*.mat` |
| Table 8, tail error of the reduction | `run_kume_e`, parts `wstail` and `domfield` | `kume_e_wstail.mat`, `kume_e_domfield.mat` |
| Numbers in Sections VI and VII | `collect_results_core.m` and the drivers above | — |


## Labels in the code

The revised paper states its results in the running text and refers to them by equation and section. Comments and printed output in the code still use the labels of the first submission; they map as follows.

| Label in the code | Location in the revised paper |
|---|---|
| Lemma 1, Definition 1, Remark 1 | Section III-A, (11)–(12) |
| Lemma 2 | Section III-B, (13) |
| Lemma 3, Remark 2 | Section III-C, (14) |
| Theorem 1, Remark 3 | Section IV-B, (17)–(18); Appendix A |
| Corollary 1 | Section IV-B, (19) |
| Corollary 2 | Section IV-B, after (19); Appendix C-A |
| Definition 2, Theorem 2 | Section IV-C, (20)–(21); Appendix C-B |
| Proposition 1, Remark 4 | Section IV-C; Appendix C-C |
| Proposition 2 | Section IV-D, (22)–(23); Appendix C-D |
| Corollary 3 | Section IV-E, (24) |
| Proposition 3 | Section IV-F |
| Theorem 3, Lemma 4 | Section V-B, (27); Appendix C-E |
| Proposition 4 | Section V-C, (29); Appendix C-F |
| Theorem 4, Remark 5 | Section V-C, (30) |
| Proposition 5 | Section V-C, (31) |
| Proposition 6, Remark 6 | Section VI-A |
| Proposition 7 | Section VI-B; Appendix C-G |
| Proposition 8 | Appendix B |
| Table II … Table VIII | Table 2 … Table 8 |

## Numerical methods

**Correlated field.** The speckle of every field is drawn over the whole aperture from the Cholesky factor of the Abdi–Kaveh kernel (1), not block by block (`fama_field.m`).

**Deep floors.** Field floors use conditional Monte Carlo: the serving textures are integrated out exactly, so a realization contributes without an outage event having to occur in it (`fama_cmc_floor.m`). The reference estimator `fama_cmc_ref.m` adds defensive importance sampling of the emitter textures for *U* ≤ 4. Every simulated value is reported with its relative standard error.

**Operating floors.** Training and selection are simulated explicitly, with all emitters active during the pilot interval. Below 10⁻³ the textures are drawn from tilted laws and reweighted (`fama_mc_select.m`, `fama_mc_sense.m`).

**Closed forms.** The alternating sum (17) cancels heavily. The code evaluates the block floor through the cancellation-free form of Section IV-B (quantile-spaced panels in the texture and Gauss–Laguerre quadrature in the interference) and checks it against high-precision references in `fama_selftest.m`.

## Citation

```bibtex
@article{Aldemir2026FAMA,
  author  = {Aldemir, Mevlana Halit and Karabulut, Muhammet Ali and Shah, A. F. M. Shahen and Madahana, Milka},
  title   = {Outage Analysis of Multi-User {FAMA} under Heterogeneous Interference over Block-Correlated Channels with {Nakagami-$m$} Textures},
  journal = {IEEE Access},
  year    = {2026},
  note    = {Revised manuscript under review}
}
```

The entry will be updated with volume, pages and DOI after publication.

## Funding

This work was supported by the Scientific and Technological Research Council of Türkiye (TÜBİTAK) under Project 124E519.

## License and contact

Released under the MIT License; see `LICENSE`.

Questions and bug reports are welcome as GitHub issues. Corresponding author: Muhammet Ali Karabulut, National Defence University, İstanbul, Türkiye (muhammetali.karabulut@msu.edu.tr).
