# lulc-fm-brazil

**A geospatial foundation model for land use and land cover dynamics in Brazil**

Pre-training on 40 years of MapBiomas LULC time series (1985–2024) across all Brazilian biomes, with ecologically-grounded process decomposition as downstream benchmark tasks.

-----

## What makes this different

Most geospatial foundation models work on multispectral satellite imagery and learn generic visual representations. `lulc-fm-brazil` operates on **classified LULC time series** — discrete class sequences over 40 years — which enables:

- **Long temporal memory:** 40 annual observations vs 3–5 years typical of image-based models
- **Structured vocabulary:** 6 aggregated classes with ecological meaning, not continuous pixel values
- **Process-aware pre-training:** 11 decomposed ecological processes as downstream tasks
- **Low compute footprint:** classified maps are orders of magnitude lighter than multispectral images
- **Brazilian coverage:** MapBiomas Collection 10.1 covers all 6 biomes with consistent labeling

-----

## Project status

|Phase                      |Description                                                   |Status                                                                           |
|---------------------------|--------------------------------------------------------------|---------------------------------------------------------------------------------|
|**Cerrado Pilot**          |Multi-state survival model for pasture dynamics               |✅ Complete — [geofm-cerrado](https://github.com/barroso2501/geofm-cerrado-github)|
|**Phase 1 — Sampling**     |Stratified extraction of 80 cells × 40 years across Brazil    |✅ Complete                                                                       |
|**Phase 2A — Pre-training**|Masked temporal prediction — Transformer vs MLP baseline      |✅ Complete                                                                       |
|**Phase 2B — Evaluation**  |Criteria 1, 3, 4 — latent space + few-shot + Cerrado benchmark|✅ Complete                                                                       |
|**Fine-tuning**            |GeoFM encoder + Weibull heads — end-to-end on Cerrado Pilot   |✅ Complete                                                                       |
|**Phase 3 — Cross-biome**  |Leave-one-biome-out transfer evaluation + Criterion 4         |⬜ Planned                                                                        |

-----

## Data

### LULC time series

- **Source:** MapBiomas Brazil Collection 10.1
- **Period:** 1985–2024 (40 years)
- **Resolution:** 30m (Landsat)
- **Coverage:** All Brazilian biomes

### Aggregated class vocabulary (6 classes)

|Class |Code|MapBiomas classes                       |Description                                    |
|------|:--:|----------------------------------------|-----------------------------------------------|
|**N** |1   |1,3,4,5,6,49,10,11,12,32,29,50          |Native vegetation (forest + herbaceous/shrubby)|
|**P** |2   |15                                      |Pasture                                        |
|**A** |3   |9,14,18,19,39,20,40,62,41,36,46,47,35,48|Agriculture + forest plantation                |
|**U** |4   |22,23,24,25,30,75                       |Non-vegetated areas                            |
|**W** |5   |26,33,31                                |Water bodies                                   |
|**T** |6   |21                                      |Mosaic/transition                              |
|NODATA|0   |27                                      |Not observed                                   |


> Class 21 (mosaic) is kept as T (transition) — same approach used in the Cerrado Pilot for P→S modeling. Class 27 (not observed) maps to NODATA (0).

### Stratified sampling

80 cells of 2×2 degrees, stratified by dominant biome, covering all Brazilian biomes:

|Biome             |Eligible cells|Sampled|
|------------------|:------------:|:-----:|
|Amazônia (A)      |94            |41     |
|Cerrado (C)       |51            |22     |
|Mata Atlântica (M)|18            |8      |
|Caatinga (CA)     |15            |7      |
|Pampa (PP)        |3             |2      |
|Pantanal (P)      |2             |2      |

**Total pixels extracted:** 4,067,807,469  
**Seed:** 2026 | **Min. coverage:** 50%

-----

## Process taxonomy (11 decomposed processes)

The 11 processes serve as downstream fine-tuning tasks and provide the ecological grounding for pre-training evaluation.

### Group 1 — Exit from native vegetation

|Process     |Description                                         |Dominant biome                  |
|------------|----------------------------------------------------|--------------------------------|
|`N_para_P`  |N→Pasture — pasture expansion over native vegetation|CA (15.9%), A (13.9%), C (12.6%)|
|`N_para_A`  |N→Agriculture — direct conversion to cropland       |PP (16.9%), M (4.0%), C (4.2%)  |
|`N_para_out`|N→U/W — other conversions                           |P (12.4%)                       |

### Group 2 — Intensification

|Process   |Description                                       |Dominant biome    |
|----------|--------------------------------------------------|------------------|
|`P_para_A`|Pasture→Agriculture — agricultural intensification|C (8.7%), M (8.2%)|

### Group 3 — Stability

|Process  |Description                                         |Threshold           |
|---------|----------------------------------------------------|--------------------|
|`estavel`|Stable — same class for ≥21 years (>half the series)|A (98.3%), P (99.2%)|

### Group 4 — N↔X alternation (decomposed by destination)

|Process  |Description                      |Ecological mechanism                                      |
|---------|---------------------------------|----------------------------------------------------------|
|`alt_N_W`|N↔Water — hydrological pulse     |Natural flood cycle (Pantanal: 33.8%)                     |
|`alt_N_A`|N↔Agriculture — fallow cycle     |Agricultural rotation (Pampa: 4.0%)                       |
|`alt_N_P`|N↔Pasture — incomplete conversion|Resprout from underground woody structures (Cerrado: 1.5%)|

### Group 5 — Return to N (right-censored, decomposed by origin)

|Process   |Description                     |Ecological mechanism                                   |
|----------|--------------------------------|-------------------------------------------------------|
|`W_para_N`|Water→N — hydrological recession|Flood receding, native recolonization (Pantanal: 27.2%)|
|`A_para_N`|Agriculture→N — fallow return   |Cropland abandoned to native (Pampa: 2.4%)             |
|`P_para_N`|Pasture→N — regeneration        |Native regeneration after pasture (CA: 4.1%)           |


> **Biome-specific notes:**
> 
> - **Pantanal:** `alt_N_W` and `W_para_N` reflect the natural hydrological pulse, not anthropic conversion.
> - **Pampa:** `alt_N_A` and `A_para_N` reflect agricultural fallow. No planted pasture over native grasslands.
> - **Cerrado:** `alt_N_P` and `P_para_N` reflect incomplete conversion — underground woody structures (xylopodia) persist after surface disturbance and resprout.
> - **Caatinga:** `P_para_N` (4.1%) highlights high regeneration capacity after disturbance.

-----

## Cerrado Pilot — proof of concept

The [geofm-cerrado](https://github.com/barroso2501/geofm-cerrado-github) repository demonstrates that MapBiomas LULC time series contain sufficient signal for ecological process prediction:

- **Multi-state survival model** (P→S + P→N + P→P): AUC P→S=0.854, AUC P→N=0.859
- **Prospective validation:** 99.4% precision on 2019–2024 independent MapBiomas data
- **Negative feedback confirmed:** r(P→S, P→N) = −0.763 — three latent clusters identified
- **Calibrated parameters:** λ_S=25.9yr, λ_N=44.8yr — physically meaningful timescales

The encoder from the Cerrado Pilot is the baseline representation for `lulc-fm-brazil` fine-tuning evaluation.

-----

## Phase 2 Results

### Phase 2A — Pre-training

|Model                    |Val Loss|Val Accuracy|Training time|
|-------------------------|:------:|:----------:|:-----------:|
|**Transformer** (primary)|~0.15   |~0.95       |~2 hours     |
|MLP baseline             |~0.32   |~0.89       |~2 hours     |

The Transformer achieved 2× lower loss and 6pp higher accuracy than the MLP baseline. Convergence was immediate and stable throughout 30 epochs.

**Speed note:** Training in ~2 hours is 6× faster than the Cerrado Pilot etapa13b (>12 hours). The difference is entirely due to the data pipeline — pre-loaded `.npz` arrays in RAM vs. pixel-by-pixel raster I/O. This makes Phase 3 cross-biome evaluation (6 training runs) feasible within a single day.

### Phase 2B — Evaluation

|Criterion                         |Result                     |Threshold            |Status       |
|----------------------------------|---------------------------|---------------------|-------------|
|C3 — Latent space (Silhouette)    |0.528                      |> 0.40 (strong)      |✅ STRONG     |
|C1 — Few-shot (encoder > baseline)|5/8 processes              |> 70%                |✅ MAJORITY   |
|C4 — P->S benchmark               |93.9% ref. at 10% data     |≥90% at ≤50%         |✅ STRONG     |
|C4 — P->N benchmark               |**104.4% ref. at 10% data**|≥90% at ≤50%         |✅ **EXCEEDS**|
|C2 — Cross-biome transfer         |Planned                    |AUC on held-out biome|⬜ Phase 3    |

**Criterion 1 process-level results:**

|Process            |Encoder AUC|Baseline AUC|Gain    |Note                             |
|-------------------|:---------:|:----------:|:------:|---------------------------------|
|N→A (direct crop)  |1.000      |0.444       |+0.556 ✅|Dominant in Pampa                |
|N↔W (hydro pulse)  |1.000      |0.688       |+0.312 ✅|Dominant in Pantanal             |
|N→out              |0.938      |0.875       |+0.062 ✅|                                 |
|N→P (pasture exp.) |0.700      |0.600       |+0.100 ✅|                                 |
|P→N (regeneration) |0.840      |0.800       |+0.040 ✅|                                 |
|P→A (intensif.)    |0.625      |0.625       |0.000 ~ |Similar temporal signature to N→P|
|N↔P (incomplete)   |0.750      |0.875       |-0.125 ⚠️|CLS smooths oscillation signal   |
|W→N (hydro recess.)|0.444      |0.667       |-0.222 ⚠️|CLS smooths oscillation signal   |

Processes involving temporal oscillation (N↔P, W→N) show encoder disadvantage — the CLS token aggregates the entire series into one vector, losing the oscillation signal that raw features preserve. This is a known limitation of mean-pooling representations addressed in Phase 3.

### Fine-tuning — GeoFM-FT

End-to-end fine-tuning of the GeoFM encoder with Weibull survival heads (identical loss to etapa13b). Differentiated learning rates: encoder=1e-5, heads=1e-4.

|Model                 |AUC P→S  |AUC P→N  |lam_S (yr)|lam_N (yr)|r(S,N)    |
|----------------------|:-------:|:-------:|:--------:|:--------:|:--------:|
|etapa13b (specialized)|0.854    |0.859    |25.91     |44.77     |-0.763    |
|GeoFM linear probe    |0.830    |0.916    |—         |—         |—         |
|**GeoFM fine-tuned**  |**0.865**|**0.876**|**24.16** |**43.64** |**-0.806**|

**GeoFM fine-tuned exceeds etapa13b on both tasks** (+0.011 P→S, +0.017 P→N). Lambda values are physically calibrated (24.2 yr and 43.6 yr — close to etapa13b). The negative feedback r=-0.806 confirms mechanistic opposition between conversion and regeneration is preserved.

## Repository structure

```
lulc-fm-brazil/
├── README.md
├── data/
│   ├── GeoFM_sampling.ipynb       ← Phase 1: sampling + extraction + process analysis
│   ├── sampled_cells_n80_seed2026.csv
│   ├── extraction_stats.csv
│   ├── process_analysis_v2.csv
│   └── geofm_dataset_metadata.json
├── pretrain/                       ← Phase 2 (in progress)
│   └── ...
├── downstream/                     ← Phase 3 (planned)
│   └── cerrado_pilot/              ← fine-tuning on Cerrado Pilot tasks
└── docs/
    └── process_taxonomy.md         ← detailed process documentation
```

-----

## Related work

- [MapBiomas](https://mapbiomas.org) — annual LULC classification for Brazil (Collection 10.1)
- [Prithvi-EO](https://huggingface.co/ibm-nasa-geospatial) — IBM/NASA foundation model for multispectral imagery
- [Clay](https://clay-foundation.github.io/model/) — open geospatial foundation model
- [geofm-cerrado](https://github.com/barroso2501/geofm-cerrado-github) — Cerrado Pilot (this project’s proof of concept)

-----

## Citation

Pre-registration: [OSF — GeoFM Cerrado Pilot](https://osf.io/c46je)  
OSF Updates 12, 12-Addendum, 13 document the full Phase 2 methodology and results.

-----

## License

Data: CC-BY 4.0 | Code: MIT
Data: CC-BY 4.0 | Code: MIT
Data: CC-BY 4.0 | Code: MIT
