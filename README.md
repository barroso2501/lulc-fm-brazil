# lulc-fm-brazil

**A geospatial foundation model for land use and land cover dynamics in Brazil**

Pre-training on 40 years of MapBiomas LULC time series (1985–2024) across all Brazilian biomes, with ecologically-grounded process decomposition as downstream benchmark tasks.

---

## What makes this different

Most geospatial foundation models work on multispectral satellite imagery and learn generic visual representations. `lulc-fm-brazil` operates on **classified LULC time series** — discrete class sequences over 40 years — which enables:

- **Long temporal memory:** 40 annual observations vs 3–5 years typical of image-based models
- **Structured vocabulary:** 6 aggregated classes with ecological meaning, not continuous pixel values
- **Process-aware pre-training:** 11 decomposed ecological processes as downstream tasks
- **Low compute footprint:** classified maps are orders of magnitude lighter than multispectral images
- **Brazilian coverage:** MapBiomas Collection 10.1 covers all 6 biomes with consistent labeling

---

## Project status

| Phase | Description | Status |
|-------|-------------|--------|
| **Cerrado Pilot** | Multi-state survival model for pasture dynamics | ✅ Complete — [geofm-cerrado](https://github.com/barroso2501/geofm-cerrado-github) |
| **Phase 1 — Sampling** | Stratified extraction of 80 cells × 40 years across Brazil | ✅ Complete |
| **Phase 2 — Pre-training** | Self-supervised pre-training objective definition and implementation | 🔄 In progress |
| **Phase 3 — Fine-tuning** | Downstream task evaluation on Cerrado Pilot benchmarks | ⬜ Planned |

---

## Data

### LULC time series

- **Source:** MapBiomas Brazil Collection 10.1
- **Period:** 1985–2024 (40 years)
- **Resolution:** 30m (Landsat)
- **Coverage:** All Brazilian biomes

### Aggregated class vocabulary (6 classes)

| Class | Code | MapBiomas classes | Description |
|-------|:----:|-------------------|-------------|
| **N** | 1 | 1,3,4,5,6,49,10,11,12,32,29,50 | Native vegetation (forest + herbaceous/shrubby) |
| **P** | 2 | 15 | Pasture |
| **A** | 3 | 9,14,18,19,39,20,40,62,41,36,46,47,35,48 | Agriculture + forest plantation |
| **U** | 4 | 22,23,24,25,30,75 | Non-vegetated areas |
| **W** | 5 | 26,33,31 | Water bodies |
| **T** | 6 | 21 | Mosaic/transition |
| NODATA | 0 | 27 | Not observed |

> Class 21 (mosaic) is kept as T (transition) — same approach used in the Cerrado Pilot for P→S modeling. Class 27 (not observed) maps to NODATA (0).

### Stratified sampling

80 cells of 2×2 degrees, stratified by dominant biome, covering all Brazilian biomes:

| Biome | Eligible cells | Sampled |
|-------|:--------------:|:-------:|
| Amazônia (A) | 94 | 41 |
| Cerrado (C) | 51 | 22 |
| Mata Atlântica (M) | 18 | 8 |
| Caatinga (CA) | 15 | 7 |
| Pampa (PP) | 3 | 2 |
| Pantanal (P) | 2 | 2 |

**Total pixels extracted:** 4,067,807,469  
**Seed:** 2026 | **Min. coverage:** 50%

---

## Process taxonomy (11 decomposed processes)

The 11 processes serve as downstream fine-tuning tasks and provide the ecological grounding for pre-training evaluation.

### Group 1 — Exit from native vegetation
| Process | Description | Dominant biome |
|---------|-------------|----------------|
| `N_para_P` | N→Pasture — pasture expansion over native vegetation | CA (15.9%), A (13.9%), C (12.6%) |
| `N_para_A` | N→Agriculture — direct conversion to cropland | PP (16.9%), M (4.0%), C (4.2%) |
| `N_para_out` | N→U/W — other conversions | P (12.4%) |

### Group 2 — Intensification
| Process | Description | Dominant biome |
|---------|-------------|----------------|
| `P_para_A` | Pasture→Agriculture — agricultural intensification | C (8.7%), M (8.2%) |

### Group 3 — Stability
| Process | Description | Threshold |
|---------|-------------|-----------|
| `estavel` | Stable — same class for ≥21 years (>half the series) | A (98.3%), P (99.2%) |

### Group 4 — N↔X alternation (decomposed by destination)
| Process | Description | Ecological mechanism |
|---------|-------------|----------------------|
| `alt_N_W` | N↔Water — hydrological pulse | Natural flood cycle (Pantanal: 33.8%) |
| `alt_N_A` | N↔Agriculture — fallow cycle | Agricultural rotation (Pampa: 4.0%) |
| `alt_N_P` | N↔Pasture — incomplete conversion | Resprout from underground woody structures (Cerrado: 1.5%) |

### Group 5 — Return to N (right-censored, decomposed by origin)
| Process | Description | Ecological mechanism |
|---------|-------------|----------------------|
| `W_para_N` | Water→N — hydrological recession | Flood receding, native recolonization (Pantanal: 27.2%) |
| `A_para_N` | Agriculture→N — fallow return | Cropland abandoned to native (Pampa: 2.4%) |
| `P_para_N` | Pasture→N — regeneration | Native regeneration after pasture (CA: 4.1%) |

> **Biome-specific notes:**
> - **Pantanal:** `alt_N_W` and `W_para_N` reflect the natural hydrological pulse, not anthropic conversion.
> - **Pampa:** `alt_N_A` and `A_para_N` reflect agricultural fallow. No planted pasture over native grasslands.
> - **Cerrado:** `alt_N_P` and `P_para_N` reflect incomplete conversion — underground woody structures (xylopodia) persist after surface disturbance and resprout.
> - **Caatinga:** `P_para_N` (4.1%) highlights high regeneration capacity after disturbance.

---

## Cerrado Pilot — proof of concept

The [geofm-cerrado](https://github.com/barroso2501/geofm-cerrado-github) repository demonstrates that MapBiomas LULC time series contain sufficient signal for ecological process prediction:

- **Multi-state survival model** (P→S + P→N + P→P): AUC P→S=0.854, AUC P→N=0.859
- **Prospective validation:** 99.4% precision on 2019–2024 independent MapBiomas data
- **Negative feedback confirmed:** r(P→S, P→N) = −0.763 — three latent clusters identified
- **Calibrated parameters:** λ_S=25.9yr, λ_N=44.8yr — physically meaningful timescales

The encoder from the Cerrado Pilot is the baseline representation for `lulc-fm-brazil` fine-tuning evaluation.

---

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

---

## Related work

- [MapBiomas](https://mapbiomas.org) — annual LULC classification for Brazil (Collection 10.1)
- [Prithvi-EO](https://huggingface.co/ibm-nasa-geospatial) — IBM/NASA foundation model for multispectral imagery
- [Clay](https://clay-foundation.github.io/model/) — open geospatial foundation model
- [geofm-cerrado](https://github.com/barroso2501/geofm-cerrado-github) — Cerrado Pilot (this project's proof of concept)

---

## Citation

Pre-registration: [OSF — GeoFM Cerrado Pilot](https://osf.io/c46je)

---

## License

Data: CC-BY 4.0 | Code: MIT
