# Multi-User-FAMA---Access-Codes
M. H. Aldemir, M. A. Karabulut, A. F. M. S. Shah and M. Madahana, “Multi-User FAMA under Heterogeneous Interference: Outage Floors over Block-Correlated Channels with Nakagami-m Textures,” IEEE Access, revised manuscript under review, 2026.
The paper studies a fluid antenna terminal that selects one of N strongly correlated ports while U − 1 co-channel emitters of unequal power, fading severity and in-aperture profile transmit, including during channel training. It derives the interference-limited outage floor in closed form, its diversity order, the effect of finite training and a parameter-free correction for the coupling between blocks. It also introduces a selector that senses the interference in the pilot residual. All analytical results are checked against a simulation of the fully correlated aperture.

Requirements
MATLAB R2021a or later, or GNU Octave 8.4 or later.
No toolboxes. Gamma variates, Tricomi's function and the Gaussian orthant probabilities are implemented in base MATLAB (fama_gamrnd.m, fama_tricomi.m, fama_mvnorthant.m).
% 1) numerical self-test (36 checks); it must pass before anything else
cd src
fama_selftest

% 2) redraw Figs. 2-5 from the saved results, without rerunning any simulation
cd ../data
addpath('../src')
plot_fig2; plot_fig3; plot_fig4; plot_fig5     % writes fig2_outage_snr.eps ... fig5_diversity.eps

% 3) full reproduction: every figure, table and number, logged to fama_log.txt
cd ../src
run_all                                          % set QUICK = true inside for a faster smoke test
The full run is dominated by run_kume_e (about 7 h of CPU time in Octave) and Table V (about 1.5 h); MATLAB is faster. QUICK = true cuts collect_results_core and run_kume_e to about 1% of their realizations, so the shapes are right but the numbers are not those of the paper; Fig. 1 and Tables II and V still run in full (table2_selectors(ntr) accepts a smaller number of runs).

A single part of a script can be run on its own, for example
part = 'wstail'; run_kume_e      % Proposition 8 and the block-model columns of Table VIII
src/     all .m files (configuration, analytical routines, estimators, figure and table scripts)
data/    saved results (MATLAB v7 .mat) from which plot_fig2 ... plot_fig5 redraw Figs. 2-5
docs/    CHANGES_KumeA.txt ... CHANGES_KumeH.txt, the code changes of each revision step
fama_config.m is the single source of truth for every parameter (Table III of the paper): N = 100 ports over W = 3λ, B = 4 blocks of L_b = 25 ports, block severities m_b = {3.0, 2.4, 1.8, 1.4}, emitter severity m_I = 2.5, per-emitter SIR 10 dB, threshold 0 dB and T_p = 16 pilot symbols.

What reproduces what
Object in the paper	Script	Saved data
Fig. 1 (block SIR distribution)	fig2_sircdf.m (writes fig1_sircdf.eps)	—
Fig. 2 (outage against pilot SNR, sensing rule)	run_kume_e, part fig2, then plot_fig2	kume_e_fig2.mat
Fig. 3 (floor against U, FGM ceiling)	run_kume_e, parts cmp and bigU, then plot_fig3	kume_e_cmp.mat, kume_e_bigU.mat
Fig. 4 (hardening, threshold sweep)	run_kume_e, part fig4, then plot_fig4	kume_e_fig4.mat
Fig. 5 (diversity on the correlated field)	fig5_diversity.m and run_kume_e, part fig5, then plot_fig5	fig5_data.mat, kume_e_fig5.mat
Table II (selectors)	table2_selectors.m	—
Table III (configuration)	fama_config.m	—
Table IV (anatomy of the gap)	run_kume_e, part cmp	kume_e_cmp.mat
Table V (training-limited floors)	table5_training.m	—
Table VI (heterogeneous emitters)	run_kume_f, part table	—
Table VII (comparison with existing analyses)	run_kume_e, parts cmp, sev, corr, transfer, oracle, summary	kume_e_*.mat
Table VIII (tail error of the reduction)	run_kume_e, parts wstail and domfield	kume_e_wstail.mat, kume_e_domfield.mat
Numbers of Sections VI and VII	collect_results_core.m and run_kume_e	—

fig3_outage_snr.m, fig4_floor_vs_U.m and fig6_hardening.m are the figure scripts of the first submission. They are kept for traceability; the revised figures come from plot_fig2 … plot_fig5.

Revision steps and reviewer comments

Each revision step has its own driver script, which prints every number that the step added to the paper.

Step	Script	Reviewer comments	Topic
A	run_kume_a.m	R1.2, R2.1	composite fading against a per-port Nakagami-m field
B	run_kume_b.m	R1.4, R2.3	scope of Theorem 1, look-count bracket (Corollary 2)
C	run_kume_c.m	R1.5, R2.4, R3.5	floor diversity order on the correlated field (Proposition 1, Fig. 5)
D	run_kume_d.m	R1.6, R2.2	interference-aware LMMSE training, training-limited floor (Proposition 2)
E	run_kume_e.m	R1.3, R1.8, R2.6, R3.6	reference estimator, tail error of the reduction (Proposition 8), Figs. 2–5
F	run_kume_f.m	R2.5	heterogeneous emitters (Proposition 4, Remark 5, Table VI)
G	run_kume_g.m	R1.1, R3.1, R3.4	copula corrections and existing analyses (Lemma 4, Section VII-I)
H	run_kume_h.m	R1.7	interference-sensing selector (Proposition 7, Tables II and V)
Numerical methods
Correlated field. The speckle of every field is drawn over the whole aperture from the Cholesky factor of the Abdi–Kaveh kernel, eq. (1), and not block by block (fama_field.m).
Deep floors. Field floors use conditional Monte Carlo: the serving textures are integrated out exactly, so a realization contributes without an outage event having to occur in it (fama_cmc_floor.m). The reference estimator fama_cmc_ref.m adds defensive importance sampling of the emitter textures for U ≤ 4. Every simulated value is reported with its relative standard error.
Operating floors. Training and selection are simulated explicitly, with all emitters active during the pilot interval. Below 10⁻³ the textures are drawn from tilted laws and reweighted (fama_mc_select.m, fama_mc_sense.m).
Closed forms. The alternating sum of Theorem 1 cancels heavily. The code evaluates it through the cancellation-free route of Remark 3 (inverse-CDF panels and Gauss–Laguerre quadrature) and checks it against high-precision references in fama_selftest.m.

@article{Aldemir2026FAMA,
  author  = {Aldemir, Mevlana Halit and Karabulut, Muhammet Ali and Shah, A. F. M. Shahen and Madahana, Milka},
  title   = {Multi-User {FAMA} under Heterogeneous Interference: Outage Floors over Block-Correlated Channels with {Nakagami-$m$} Textures},
  journal = {IEEE Access},
  year    = {2026},
  note    = {Revised manuscript under review}
}

The entry will be updated with volume, pages and DOI after publication.

Funding

This work was supported by the Scientific and Technological Research Council of Türkiye (TÜBİTAK) under Project 124E519.

License and contact

Released under the MIT License; see LICENSE.

Questions and bug reports are welcome as GitHub issues. Corresponding author: Muhammet Ali Karabulut, National Defence University, İstanbul, Türkiye (muhammetali.karabulut@msu.edu.tr).
