# TESS Transit Pipeline

Independent batman transit model fitting and flux dilution analysis for TESS Objects of Interest (TOIs).

**Author:** Suyasha Ghimire
**Status:** Active research - submitted to Research Notes of the AAS (RNAAS)

---

## What this does

TESS finds exoplanet candidates by watching for the brightness dip a planet causes when it crosses in front of its star. NASA's SPOC pipeline measures these dips and publishes a transit depth for each candidate. This project independently re-measures those same transit depths using the `batman` transit model and compares the results against the official SPOC catalog.

TESS pixels are large (21 arcseconds), so light from nearby stars often bleeds into the same aperture as the target, diluting the transit signal. The core question this pipeline investigates: do independently measured depths disagree with SPOC's published values, and if so, is flux dilution from nearby stars the cause?

## Key result

Across 18 TESS planet candidates:

- Batman-fitted depths are systematically **17.9% shallower** than SPOC catalog values (15 of 18 targets, sign test p = 0.004)
- **TOI 6449.01** has zero nearby contaminating stars and reproduces the SPOC depth to within 5% validating the pipeline
- **TOI 6022.01** shows a depth 6.7× shallower than its catalog value, inconsistent with its measured contamination, likely a background eclipsing binary misclassified as a planet candidate, identified entirely from archival photometry with no telescope time
- Contamination measured within 21 arcseconds does not significantly correlate with the depth offset (Spearman ρ = 0.17, p = 0.50), suggesting the true TESS photometric aperture (~80–100 arcseconds) extends well beyond the single-pixel search radius used here

Full write-up submitted as a Research Note to RNAAS.

## Pipeline steps

1. Download the TOI catalog from the NASA Exoplanet Archive
2. Filter to well-characterized planet candidates (PC/APC disposition, 1–10 day period, depth > 500 ppm)
3. For each target:
   - Query the TIC for stellar effective temperature → fixed limb-darkening coefficients (Claret 2017)
   - Query the TIC for sources within 21 arcseconds → contamination ratio and dilution factor
   - Download SPOC 2-minute light curves from MAST
   - Detrend with a Savitzky-Golay filter (1.5-day window)
   - Refine the orbital period with a Box Least Squares search
   - Phase-fold and bin the light curve
   - Fit a 4-parameter batman transit model (t₀, Rp/Rs, a/Rs, inclination)
4. Compare fitted depths against SPOC catalog values
5. Plot contamination ratio vs. depth ratio to test the flux dilution hypothesis

## Installation

```bash
git clone https://github.com/soyapoyaa/tess-transit-pipeline.git
cd tess-transit-pipeline
pip install lightkurve batman-package astroquery scipy pandas matplotlib astropy
```

Python 3.9+

## Usage

```bash
jupyter notebook toi_batman_pipeline.ipynb
```

Run the notebook top to bottom. Change `n_targets` in the batch run cell to control how many TOIs are analyzed, start with 5 to verify the pipeline, then scale up.

## Output

All results are saved to `results/`:

| File | Description |
|---|---|
| `toi_population.png` | Period, depth, and SNR distributions of the candidate pool |
| `TOI_XXXX_batman.png` | 4-panel diagnostic figure for each individual target |
| `catalog_vs_fitted_depth.png` | Batman-fitted depths vs. SPOC catalog depths |
| `contamination_vs_depth_ratio.png` | Main result: dilution analysis across all targets |
| `toi_batman_results.csv` | Full results table with all fitted parameters |

## Limitations

- The 21-arcsecond contamination query is smaller than the typical TESS photometric aperture (~80–100 arcseconds), which likely explains the weak contamination–depth correlation
- Nearly all targets are M dwarfs (Teff ≈ 3200–3500 K), where the Claret (2017) limb-darkening tables are less reliable
- Savitzky-Golay window sensitivity and period-smearing across sectors were not independently quantified
- TOI 6022.01's eclipsing binary classification is based on depth discrepancy alone; standard vetting diagnostics (odd-even test, secondary eclipse search, centroid analysis) have not yet been applied

## References

- Claret, A. 2017, A&A, 600, A30
- Giacalone, S., Dressing, C. D., Jensen, E. L. N., et al. 2021, AJ, 161, 24
- Jenkins, J. M., Twicken, J. D., McCauliff, S., et al. 2016, Proc. SPIE, 9913, 99133E
- Kostov, V. B., Schlieder, J. E., Barclay, T., et al. 2019, AJ, 158, 32
- Kovács, G., Zucker, S., & Mazeh, T. 2002, A&A, 391, 369
- Kreidberg, L. 2015, PASP, 127, 1161
- Lightkurve Collaboration, Cardoso, J. V. d. M., Hedges, C., et al. 2018, ASCL, ascl:1812.013
- Mandel, K., & Agol, E. 2002, ApJ, 580, L171
- Ricker, G. R., Winn, J. N., Vanderspek, R., et al. 2015, JATIS, 1, 014003
- Southworth, J. 2008, MNRAS, 386, 1644

## License

MIT
