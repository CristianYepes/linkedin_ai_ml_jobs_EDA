# EDA – AI/ML Job Listings (USA)

**Dataset:** `kanchana1990/ai-and-ml-job-listings-usa` → `ai_ml_jobs_linkedin.csv`
**Objetivo:** entender calidad de datos y factores asociados al interés (nº de aplicaciones por oferta).
**Nota clave:** el conteo está **censurado** (“Over 200 applicants”) → se usa también **apps_per_day**.

---

## 1) Flujo y metodología
1. **Carga & QC:** verificación tabular, muestra `preview_100_rows.csv`, nulos y duplicados.
2. **Limpieza y tipificación:**
   - `publishedAt` → `datetime`.
   - `applicationsCount` → **`applicationsCount_num`** (extracción numérica de textos).
   - Strings normalizados (trim), categorías de baja cardinalidad como `category`.
3. **Feature engineering:**
   - **Target recomendado:** `apps_per_day = applicationsCount_num / días_desde_publishedAt`.
   - **Censura:** `is_top200 = 1{applicationsCount_num ≥ 200}`.
   - **Texto:** `title_len`, `desc_len`; variantes `log1p_*` y winsor p99 (solo para análisis/modelado).
   - **Ubicación/Sector:** `state` desde `location`; `sector_main` (primera etiqueta de `sector`); `location_clean` sin valores genéricos.
4. **Análisis:**
   - **Univariable:** histogramas y boxplots robustos (cap p1–p99; log1p si hay sesgo).
   - **Categóricas vs target (Paso 7):** barras horizontales por **mediana** y **% al tope (≥200)** con `min_count ≥ 10`.
   - **Outliers (Paso 8):** IQR/Z + visual robusta.
   - **Correlaciones (Paso 9):** Spearman/Kendall y scatter con recorte p1–p99.

---

## 2) Calidad de datos
- **Censura a la derecha** en `applicationsCount_num` (muchos registros en **200**).
- Picos también cerca de **25** (frases tipo “Be among the first 25 applicants”).
- `location` mezcla granularidades (país/ciudad) → se emplean `state` y `location_clean`.
- Duplicados y nulos presentes pero no bloqueantes para EDA (se reportaron en el notebook).

---

## 3) Variables y transformaciones
- **Targets:**
  - `applicationsCount_num` (crudo, censurado).
  - **`apps_per_day`** (recomendado para comparar ofertas publicadas en fechas distintas).
  - `is_top200` (binario para analizar proporciones en el tope).
- **Texto:** `title_len`, `desc_len` → muy asimétricas; se recomienda `log1p_*` o winsor p99.
- **Categóricas clave:** `experienceLevel`, `workType`, `contractType`, `state`, `sector_main`, `companyName`, `title`.

---

## 4) Univariable (resumen)
- `applicationsCount_num`: bimodal (≈25 y **200**).
- `apps_per_day`: distribución más compacta; útil para comparar “tasa” de interés.
- `title_len` y `desc_len`: colas largas y outliers; usar escala log para visual/modelado.

---

## 5) Target ~ Categóricas (Paso 7)
- Para cada categórica (Top-20 con `n ≥ 10`) se graficó:
  - **Mediana de `applicationsCount_num`** (barras horizontales limpias).
  - **% de ofertas al tope (≥200)**.
- **Lectura cualitativa:**
  - `state`/`sector_main` muestran diferencias entre categorías, aunque con **alta dispersión** entre ofertas.
  - `experienceLevel`: patrón razonable (Entry > Senior en interés), con solape; mirar mediana + % al tope.
  - `contractType` aporta poco si el 90%+ es Full-time.
  - `companyName`: interpretar solo con suficiente `n` (p. ej., `n ≥ 10`).

---

## 6) Outliers (Paso 8)
- Alto % de extremos en longitudes (`title_len`, `desc_len`) por colas largas.
- Visualización **robusta**: cap p1–p99 y boxplots sin *fliers*; histogramas log1p cuando hay sesgo.
- Recomendación: trabajar con `log1p_desc_len`, `log1p_title_len` y/o winsor p99 para modelos.

---

## 7) Correlaciones y pares numéricos (Paso 9)
**Spearman (RAW):**
- `applicationsCount_num` ~ `apps_per_day`: **0.9405** (muy alta, por construcción).
- `title_len` ~ `desc_len`: **0.2114** (débil–moderada).
- `applicationsCount_num` ~ `desc_len`: **0.1395** (débil).
- `applicationsCount_num` ~ `title_len`: **0.0243** (casi nula).

**Kendall (RAW):**
- `applicationsCount_num` ~ `apps_per_day`: **0.8545**.
- `title_len` ~ `desc_len`: **0.1459**.

**LOG1P (al vuelo):** igual patrón (Spearman/Kendall son de rangos).
**Implicación:** no usar simultáneamente `applicationsCount_num` y `apps_per_day` (colinealidad). Longitudes explican poco el interés; la señal útil está más en **categóricas**.

---
