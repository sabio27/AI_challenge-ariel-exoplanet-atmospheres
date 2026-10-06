# Water and clouds in exoplanet atmospheres

Team entry (UFHB LightWaves) to the **Défi IA 2025** organised around the ESA **Ariel** mission, which will observe the atmospheres of about 1,000 exoplanets from 2029. We finished **3rd nationally**, with a score of 0.974 on the competition leaderboard.

## The problem

When a planet passes in front of its star, part of the starlight goes through the planet's atmosphere. How much light is blocked at each wavelength (the transit depth) depends on what the atmosphere contains. The task was to tell, from a simulated Ariel spectrum, whether the atmosphere contains **water** and whether it has **clouds**: two yes/no questions per planet.

Each planet comes with:

- a spectrum of 52 points between 0.5 and 7.8 µm (wavelength, transit depth, uncertainty), covering Ariel's two instruments, FGS and AIRS;
- a few physical parameters: star mass, radius and temperature, planet mass, orbit size.

There are 3,000 labelled planets for training and 1,032 to predict. Both classes are balanced (50% with water, 50% with clouds).

## Approach

**Features (80 per planet).**
- The 52 spectrum values, normalised planet by planet (median and median absolute deviation). This removes the effect of the planet's size, which shifts the whole spectrum up or down.
- 17 statistics computed on each spectrum: mean, spread, skewness and kurtosis, gap between the FGS and AIRS bands, depth in the 1.1–1.6 µm water band, signal-to-noise ratio, slope and curvature, entropy, autocorrelation, number of peaks.
- 11 physical quantities derived from the system parameters: masses and radii in solar and Jovian units, theoretical planet radius, equilibrium temperature, effective gravity, insolation, atmospheric scale height.

**Models.** A GradientBoosting model (scikit-learn) and a LightGBM model for each target, with their probabilities averaged.

**Validation.** 5-fold cross-validation stratified on the water/clouds combination. Test predictions are averaged over the 5 folds. The decision threshold for each target is chosen on the out-of-fold predictions.

## Results

Out-of-fold accuracy over the 3,000 training planets:

| Target | Threshold | Accuracy |
| --- | --- | --- |
| Water | 0.37 | 98.1% |
| Clouds | 0.47 | 97.5% |
| **Mean** | | **97.8%** |

On the test set, the competition leaderboard gave a score of **0.974**, close to the cross-validation estimate.

Clouds are a little harder than water. A thick cloud layer flattens the spectrum, which can hide the water features too.

## Run it

```bash
pip install -r requirements.txt
```

Put the competition files (`spectra.npy`, `spectra_test.npy`, `auxiliary.csv`, `auxiliary_test.csv`, `targets.csv`) in `data/`. They are not included in this repository. Then run `notebooks/defi_ia_2025.ipynb`. It takes a few minutes on a laptop CPU and writes `submission.csv` at the root.

## Tools

Python, NumPy, pandas, SciPy, scikit-learn, LightGBM, matplotlib.

---

## Résumé en français

Participation de l'équipe UFHB LightWaves au **Défi IA 2025** autour de la mission Ariel de l'ESA : 3ᵉ place nationale, score de 0,974.

À partir du spectre de transit simulé d'une exoplanète et des paramètres de son système, il faut prédire la présence d'eau et de nuages dans son atmosphère. Notre solution combine :
- 80 variables : le spectre normalisé, des statistiques sur le spectre et des grandeurs physiques ;
- deux modèles de boosting (GradientBoosting et LightGBM) dont les probabilités sont moyennées ;
- un seuil de décision optimisé pour chaque cible.

En validation croisée sur 5 plis, l'accuracy moyenne est de 97,8 %.
