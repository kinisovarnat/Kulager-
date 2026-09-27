[Русский](README.md) | [Қазақша](README.kk.md) | **English**

# Kulager (Құлагер)

### A hybrid model for predicting uncontrolled re-entry of space objects

**Author:** Arnat
**Competition:** University Competition of Scientific and Innovative Projects, 2026
**Field:** aerospace engineering · space safety · machine learning
**Type:** digital product (model + web application)

---

## Summary

**About the name:** Kulager (Құлагер) is the legendary racehorse of the poet Akan Seri, celebrated in Ilyas Zhansugurov's poem. It stands for speed and precision, and it marks the project as made in Kazakhstan.

Kulager predicts when a defunct satellite, rocket stage or debris fragment will make an uncontrolled re-entry into Earth's atmosphere. The project combines a physics-based atmospheric drag model with machine learning trained on the real history of objects that have already re-entered. Accuracy is tested on real 2026 re-entries and compared with official predictions.

---

## Meeting the Competition Criteria

| Criterion | How the project meets it |
|---|---|
| **A specific problem** | About 70% of re-entries of large objects are uncontrolled. An error of a few hours in the predicted date produces a possible impact zone thousands of kilometres long. See "Problem" |
| **Novelty** | A hybrid scheme: ML predicts the error of the physics model, accounts for space weather and outputs an uncertainty interval for each object type. Plus an open tool with a map. See "Novelty" |
| **A verifiable result** | Prediction error (%) on 2026 objects the model has never seen, compared with the physics model and official TIP predictions. Data and code are open, so anyone can reproduce the result. See "How to Verify the Result" |

---

## Problem

- On average, more than three intact satellites or rocket bodies re-enter the atmosphere every day ([ESA Space Environment Report 2026](https://www.esa.int/Space_Safety/Space_Debris/ESA_Space_Environment_Report_2026)).
- About 70% of re-entries of large objects are uncontrolled, roughly 100 tonnes per year ([CNR](https://iris.cnr.it/retrieve/39fb2ff7-ecb6-45a3-a789-d9614299e934/prod_424423-doc_159798.pdf)).
- 10–40% of an object's mass survives re-entry and reaches the ground ([IAA](https://iaaspace.org/wp-content/uploads/iaa/Scientific%20Activity/debris6.pdf)).
- The risk to aircraft in flight is growing ([PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC11757734/)).

**The core problem:** upper-atmosphere density depends strongly on solar activity, so classical calculations get the re-entry date wrong. A large timing error means a large risk zone for people and aviation.

## Goal

To build a hybrid model that predicts the date of uncontrolled re-entry more accurately than a baseline physics model, and an open web tool that visualises the prediction and the risk zone.

## Novelty

1. **Hybrid approach.** ML does not replace physics; it corrects the physics model's systematic error. The model stays physically meaningful and works with a small amount of data.
2. **Space weather.** The model uses solar (F10.7) and geomagnetic (Ap) activity indices, including how they have changed over recent days.
3. **Uncertainty interval.** The model outputs a range rather than a single date, separately for satellites, rocket stages and debris.
4. **Open tool.** A free web application with a map. Such predictions are currently published mainly by specialised agencies.

Similar ML approaches to orbits already exist, for example [OrbitFM](https://huggingface.co/postcn/OrbitFM). This project differs in its "physics + error correction" scheme, its per-object-type interval forecasts and its open visualisation.

---

## How to Verify the Result

**Principle:** the model is trained on objects that re-entered in 2023–2025 and tested on 2026 objects it has never seen. The actual re-entry dates are known, so prediction accuracy can be checked objectively.

**Metric:**

```
error = |predicted − actual| / actual × 100%
```

The error is calculated separately for predictions made 30, 7 and 1 day before re-entry.

**Results table** (to be filled in after the experiments):

| Method | Error at 30 days | Error at 7 days | Error at 1 day |
|---|---|---|---|
| Baseline physics model | — | — | — |
| **Kulager (hybrid)** | — | — | — |
| Official TIP prediction (Space-Track) | — | — | — |

**Success criterion:** Kulager has a lower error than the baseline physics model at every forecast horizon.

**Reproducibility:**
- all data are open (see "Data");
- the sample is fixed (`random_state=42`) and the selection criteria are stated explicitly;
- every experiment is recorded in `experiments.csv`, including unsuccessful ones;
- day-by-day progress is recorded in `journal.md`.

---

## Project Status

- [x] Topic, data sources, methodology
- [x] Project structure, data collection notebooks (01–03)
- [ ] CelesTrak and Space-Track data collection
- [ ] Physics model (04)
- [ ] Hybrid LightGBM model (05)
- [ ] Testing on 2026 data, comparison with TIP
- [ ] Web application with a map
- [ ] Final presentation

## Work Plan

| Dates | Stage |
|---|---|
| 27 Sep – 1 Oct | Data collection and preparation |
| 2 – 4 Oct | Physics model |
| 5 – 7 Oct | Hybrid model |
| 8 – 11 Oct | Testing, comparison with TIP, charts |
| 12 – 15 Oct | Web application |
| 16 – 18 Oct | Application and presentation |

---

## Data

| Data | Source |
|---|---|
| Object catalogue and actual re-entry dates | [CelesTrak SATCAT](https://celestrak.org/pub/satcat.csv) or [Space-Track](https://www.space-track.org), `satcat` class |
| Orbital history (TLE / OMM) | [Space-Track](https://www.space-track.org), `gp_history` class |
| Official re-entry predictions | [Space-Track](https://www.space-track.org), `TIP` class |
| Solar and geomagnetic activity (F10.7, Ap, Kp) | [GFZ Potsdam](https://kp.gfz.de/app/files/Kp_ap_Ap_SN_F107_since_1932.txt) (primary, CC BY 4.0); [CelesTrak](https://celestrak.org/SpaceData/SW-All.csv) (fallback) |
| Actual re-entries (additional check) | [Aerospace CORDS](https://aerospace.org/reentries) |
| NRLMSIS atmosphere model | [pymsis](https://pypi.org/project/pymsis/) |

**Selection criteria:** objects in Earth orbit that re-entered in 2023–2026, up to 100 objects of each type (satellites, rocket stages, debris). Objects with controlled de-orbits (Starlink, Progress, Dragon, Cygnus, etc.) are excluded, because the project studies uncontrolled re-entries only.

## Methodology

1. **Features** at each point in time: perigee and apogee altitude, orbital decay rate over 3 and 7 days, B*, inclination, F10.7, Ap.
2. **Target:** days remaining until re-entry.
3. **Physics model:** orbital decay under atmospheric drag, with density from NRLMSIS.
4. **Hybrid model:** LightGBM predicts a correction to the physics forecast.
5. **Data split:** by re-entry year. An object never appears in both training and testing.

---

## Repository Structure

```
data/raw/          raw files (not included in the repository)
data/processed/    prepared data
notebooks/
  01_celestrak_data.ipynb     object catalogue and space weather
  02_spacetrack_tle.ipynb     orbital history
  03_build_dataset.ipynb      training dataset
models/            saved models
results/           tables and charts
journal.md         project journal
experiments.csv    experiment log
PROJECT_DESCRIPTION.md        full project description
```

## How to Run

1. Upload the project folder to Google Drive: `MyDrive/reentry-project`.
2. Open the notebooks in [Google Colab](https://colab.research.google.com) in order: 01 → 02 → 03.
3. A free [Space-Track](https://www.space-track.org) account is required: always for notebook 02, and for notebook 01 if CelesTrak is unreachable from Colab (a common situation; the notebook switches to Space-Track automatically).

Tools used: Python, pandas, numpy, requests, matplotlib, sgp4, pymsis, lightgbm, scikit-learn (see `requirements.txt`). The project budget is 0 tenge; all tools and data are free.

---

## Licence and Data

The code is released under the MIT Licence (see `LICENSE`).

Data are not included in the repository; notebooks 01–02 download them.
- Orbital data (TLE/OMM, SATCAT, re-entry data) are provided by USSPACECOM via [www.space-track.org](https://www.space-track.org). Space-Track permits redistribution of these data and publication of analyses based on them, provided the source is properly cited ([Space-Track documentation](https://www.space-track.org/documentation)).
- Catalogue: [CelesTrak](https://celestrak.org).
- Geomagnetic indices Kp/Ap and F10.7: GFZ Helmholtz Centre for Geosciences, CC BY 4.0. Citation: Matzka et al. (2021), Space Weather, https://doi.org/10.1029/2020SW002641
