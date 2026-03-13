# Spectral Stacking

This repository contains exploratory notebooks for stacking JADES NIRSpec/PRISM spectra. The notebook `Spectral Stacking JADES DR4 - DR5 Morphology Optical.ipynb` asks whether the rest-frame optical stacked spectrum changes with galaxy morphology at high redshift.

## What This Notebook Actually Uses

Despite the filename, the saved workflow is built from DR4 spectroscopy plus DR4 morphology and stellar-property crossmatches:

- `DR4/Combined_DR4_external_v1.2.1.fits`
- `Crossmatched_Tables/DR4_F115W_bestfit_stellar_prop.csv`
- `Crossmatched_Tables/DR4_F444W_bestfit.csv`
- PRISM 1D extractions in `DR4/goods-*/clear-prism/...`

The F444W table is loaded, but the downstream sample selection in this notebook is based on the F115W crossmatch.

## Saved-Run Sample Construction

The notebook builds the science sample in stages:

- starts from `5190` DR4 sources in the master catalog
- loads `4647` F115W crossmatched sources and `4927` F444W crossmatched sources
- removes sources with unreliable spectroscopic flags (`z_Spec_flag` equal to `D` or `E`)
- builds a combined `[O III] 5007` flux using the doublet measurement when available, otherwise the single-line measurement
- requires finite `[O III] 5007` flux and error
- removes a hand-curated list of incomplete spectra
- gathers `5106` GOODS-N and GOODS-S PRISM 1D files
- identifies `54` duplicate NIRSpec IDs and excludes them
- matches the cleaned catalog to unique PRISM files, leaving `2877` matched spectra
- applies the science cut `4 <= z <= 7`, leaving `620` galaxies

Physical sizes are computed from `R_EFF` using the Planck18 cosmology. In the saved run, the `4 <= z <= 7` sample spans roughly `86.7` to `7886` pc in physical radius, with a mean of about `1027` pc.

## Morphology Definition

The notebook follows a mass-corrected size metric of the form

`log10(R_eff / (M_star / 1e8 Msun)^0.19)`.

It then bins the sample in redshift and uses the 16th and 84th percentile envelope of this size distribution to classify galaxies as:

- `100` compact
- `420` intermediate
- `100` extended

The saved output reports a size-redshift trend of `alpha_z = -1.20 +/- 0.26`.

## Stacking Workflow

For each morphology bin, the notebook:

- reads the `EXTRACT3PIX1D` spectrum from FITS extension `2`
- shifts the spectrum to the rest frame using `z_Spec_1`
- builds a common rest-frame wavelength grid from `Line Spread Functions/meanlsf_clear_prism.csv`
- normalizes each spectrum with a linear continuum fit over `0.20-0.22 um`, evaluated at `0.21 um`
- keeps the optical analysis window `0.3-0.7 um`
- builds a median stack with uncertainties from `1000` Monte Carlo perturbations
- builds an inverse-variance-weighted mean stack after `5 sigma` clipping

The stacked optical spectra are fit with LSF-aware Gaussian emission-line models covering:

- `[Ne V]`
- `[O II]`
- `[Ne III]`
- `He I`
- `H epsilon`
- `H delta`
- `H gamma`
- `[O III] 4363`
- `H beta`
- `[O III] 4959, 5007`
- `H alpha`
- `[N II]`

The final notebook figures compare compact, extended, and intermediate optical stacks in both median and mean form, with residual panels under each fit.

## Notes And Caveats

- The notebook saves an empty `bad_sources` list in the saved run.
- The `weird_sources` section looks for compact objects at `7 < z <= 9`, but that selection is empty once the notebook applies `4 <= z <= 7`.
- The active optical continuum function used in the fit behaves as a single power law, even though the fit interface still carries separate blue and red continuum parameters.
- The notebook writes `JADES Emission Lines/new_lines_table_edit_urvi.csv` as a side effect.
- The last UV comparison cells plot UV stack variables such as `wave_uv_compact`, `model_uv_mean_compact`, and `LAM_NIV` that are not defined inside this notebook. Those saved figures therefore depend on prior kernel state or on work from the UV notebook, and are not self-contained.
