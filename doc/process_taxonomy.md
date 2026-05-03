# Process taxonomy

Detailed documentation of the 11 ecological processes used as downstream benchmark tasks in `lulc-fm-brazil`.

---

## Design principles

The process taxonomy was designed with three constraints:

**Ecologically meaningful decomposition.** Each process has a distinct ecological mechanism, a dominant biome, and a clear definition in terms of the 6-class LULC vocabulary. Generic categories like "N alternation with other classes" were decomposed into mechanistically distinct processes (hydrological pulse, fallow cycle, incomplete conversion).

**Derivable from LULC time series alone.** No external covariates are required. All 11 processes are identified from the class sequence over 1985–2024.

**Right-censoring aware.** Processes in Group 5 (return to N) are modeled as right-censored survival outcomes — the pixel has not yet completed the transition by 2024. This avoids the temporal sensitivity problem of threshold-based labels.

---

## Class vocabulary

```
N = 1  (native vegetation)
P = 2  (pasture)
A = 3  (agriculture + forest plantation)
U = 4  (non-vegetated)
W = 5  (water)
T = 6  (mosaic/transition — class 21, treated as intermediate)
0      (NODATA — class 27, not observed)
```

---

## Process definitions

### Group 1 — Exit from native vegetation

**`N_para_P` — N→Pasture**
- Definition: pixel starts in N; first non-N, non-T class is P
- Ecological meaning: pasture expansion over native vegetation
- Dominant biomes: Caatinga (15.9%), Amazônia (13.9%), Cerrado (12.6%)
- Note: the dominant pathway of deforestation across most biomes

**`N_para_A` — N→Agriculture**
- Definition: pixel starts in N; first non-N, non-T class is A
- Ecological meaning: direct conversion to cropland without pasture phase
- Dominant biomes: Pampa (16.9%), Mata Atlântica (4.0%), Cerrado (4.2%)
- Note: in Pampa, native grasslands (campo nativo) are converted directly to soybean/wheat without planted pasture phase

**`N_para_out` — N→U/W**
- Definition: pixel starts in N; first non-N, non-T class is U or W
- Ecological meaning: urbanization, mining, reservoir flooding, or other non-agricultural conversion
- Dominant biome: Pantanal (12.4%) — reservoir flooding and infrastructure

---

### Group 2 — Intensification

**`P_para_A` — Pasture→Agriculture**
- Definition: pixel is in P; next non-P, non-T class is A
- Ecological meaning: agricultural intensification — pasture converted to cropland
- Dominant biomes: Cerrado (8.7%), Mata Atlântica (8.2%)
- Note: the primary pathway of soybean expansion in the Cerrado (MATOPIBA region)

---

### Group 3 — Stability

**`estavel` — Stable**
- Definition: pixel remains in the same class (N, P, A, U, or W) for ≥21 years
- Threshold: 21 years = more than half of the 40-year series
- Ecological meaning: structurally consolidated land use — resistant to transitions
- Class T (mosaic) does not count toward stability
- Mean frequency: 94.1% of pixels across all biomes (Pantanal: 99.2%, Mata Atlântica: 85.5%)

---

### Group 4 — N↔X alternation (decomposed by destination)

Alternation is defined as ≥3 transitions between N and class X (≥1.5 complete cycles). Class T is ignored in transition counting.

**`alt_N_W` — N↔Water (hydrological pulse)**
- Definition: ≥3 transitions between N and W in the series
- Ecological mechanism: natural flood-recession cycle — native vegetation submerges during flood peak and re-emerges during recession
- Dominant biome: Pantanal (33.8%)
- Important: this is NOT anthropic conversion. It reflects the Pantanal's annual hydrological pulse, which moves large areas between flooded (W) and vegetated (N) states.

**`alt_N_A` — N↔Agriculture (fallow cycle)**
- Definition: ≥3 transitions between N and A in the series
- Ecological mechanism: cropland cultivation followed by field abandonment, with native vegetation recovery during fallow periods
- Dominant biome: Pampa (4.0%)
- Important: in Pampa, native grasslands (campo nativo) are used in rotation with annual crops. The "native" periods in the cycle represent genuine field abandonment, not planted pasture.

**`alt_N_P` — N↔Pasture (incomplete conversion)**
- Definition: ≥3 transitions between N and P in the series
- Ecological mechanism: pasture establishment without complete root removal (destoca), enabling resprout of native woody vegetation from underground structures (xylopodia in Cerrado)
- Dominant biomes: Caatinga (2.1%), Cerrado (1.5%)
- Note: this process is the surface expression of the competition between pasture management and native regeneration. Pixels oscillating between N and P have intact belowground woody biomass.

---

### Group 5 — Return to N (right-censored, decomposed by origin)

These processes identify pixels where native vegetation appears at the end of the series but not the beginning — the pixel transitioned away from N at some point and returned. They are modeled as right-censored outcomes because the transition may continue beyond 2024.

The "origin" is defined as the last non-N, non-T class before the terminal N sequence.

**`W_para_N` — Water→N (hydrological recession)**
- Definition: pixel ends in N; last class before the terminal N sequence is W
- Ecological mechanism: permanent or semi-permanent water body receding, with native vegetation recolonizing the exposed substrate
- Dominant biome: Pantanal (27.2%)
- Note: in Pantanal, this reflects long-term hydrological changes (multi-year drought cycles), distinct from the annual `alt_N_W` pulse.

**`A_para_N` — Agriculture→N (fallow return)**
- Definition: pixel ends in N; last class before the terminal N sequence is A
- Ecological mechanism: cropland permanently abandoned, with native vegetation recovering
- Dominant biomes: Cerrado (8.7% of cells reporting this process), Mata Atlântica (8.2%)
- Note: distinct from `alt_N_A` (which requires cycling) — this is a one-way return without subsequent reconversion in the observed period.

**`P_para_N` — Pasture→N (regeneration)**
- Definition: pixel ends in N; last class before the terminal N sequence is P
- Ecological mechanism: native vegetation regenerating after pasture abandonment
- Dominant biomes: Caatinga (4.1%), Mata Atlântica (2.6%), Cerrado (1.7%)
- Note: in Cerrado, this is facilitated by underground woody structures (xylopodia) that persist through pasture phase. In Caatinga, it reflects the high resilience of caatinga vegetation to disturbance.

---

## Frequency summary (mean % per cell by biome)

| Process | A | C | CA | M | P | PP |
|---------|:-:|:-:|:--:|:-:|:-:|:--:|
| N_para_P | 13.9 | 12.6 | 15.9 | 4.7 | 6.2 | 0.0 |
| N_para_A | 0.9 | 4.2 | 1.0 | 4.0 | 0.0 | **16.9** |
| N_para_out | 1.4 | 1.3 | 0.6 | 0.5 | **12.4** | 1.6 |
| P_para_A | 2.9 | **8.7** | 2.6 | **8.2** | 0.5 | 0.0 |
| estavel | **98.3** | 91.6 | 87.1 | 85.5 | **99.2** | 95.2 |
| alt_N_W | 1.2 | 0.8 | 0.2 | 0.1 | **33.8** | 0.5 |
| alt_N_A | 0.0 | 0.0 | 0.0 | 0.6 | 0.0 | **4.0** |
| alt_N_P | 1.4 | 1.5 | **2.1** | 0.3 | 1.0 | 0.0 |
| W_para_N | 0.3 | 0.6 | 0.2 | 0.2 | **27.2** | 0.6 |
| A_para_N | 0.0 | 0.2 | 0.0 | 0.6 | 0.0 | 2.4 |
| P_para_N | 0.3 | 1.7 | **4.1** | 2.6 | 0.7 | 0.0 |

Bold = dominant biome for that process.
Biomes: A=Amazônia, C=Cerrado, CA=Caatinga, M=Mata Atlântica, P=Pantanal, PP=Pampa.

---

## Implementation

See `data/GeoFM_sampling.ipynb` — function `classificar_pixel()` for the reference implementation of all 11 processes.

Stability threshold: `THRESHOLD_ESTAB = 21` years.
Alternation threshold: ≥3 transitions between N and class X.
