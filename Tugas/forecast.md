# Skforecast — Model Explainability

**Reproduksi dari:** [https://skforecast.org/0.15.1/user_guides/explainability.html](https://skforecast.org/0.15.1/user_guides/explainability.html)

Artikel ini mendemonstrasikan teknik-teknik **explainability** pada model forecasting berbasis machine learning menggunakan library **skforecast**. Topik yang dicakup:

- Feature Importance (bawaan model)
- Permutation Importance
- SHAP Values (Global & Local)
- Partial Dependence Plots (PDP)

---

## Jawaban Pertanyaan Tugas

**1. Analisa prediksi tentang apa?**
Memprediksi **permintaan listrik (electricity demand)** harian di Victoria, Australia menggunakan model LightGBM yang dibalut dalam `ForecasterRecursive` dari skforecast.

**2. Bentuk data training (input & output)?**
- **Input (X):** Nilai-nilai lag (`lag_1` s/d `lag_7`) yaitu konsumsi listrik hari-hari sebelumnya, ditambah variabel eksogen (Temperature)
- **Output (y):** Nilai `Demand` pada hari berikutnya yang ingin diprediksi

**3. Apa itu lag?**
Lag adalah nilai time series pada waktu sebelumnya yang dijadikan fitur input. `lag_1` = nilai 1 hari lalu, `lag_2` = 2 hari lalu, dst. Mekanisme lag inilah yang mengubah masalah time series menjadi masalah *supervised learning* biasa.

**4. Proses analisis:**
Load data → Agregasi harian → Split train/test → Buat & latih Forecaster → Ekstrak matriks training → Feature Importance → Permutation Importance → SHAP → Partial Dependence Plot

---

## Instalasi Library

```bash
pip install skforecast==0.15.1 lightgbm shap
```

---

## Import Libraries

```code-cell
# Libraries
# ==============================================================================
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import shap
from sklearn.inspection import permutation_importance
from sklearn.inspection import PartialDependenceDisplay
from lightgbm import LGBMRegressor
from skforecast.datasets import fetch_dataset
from skforecast.recursive import ForecasterRecursive

shap.initjs()  # Inisialisasi JavaScript untuk visualisasi SHAP di notebook
print('✅ Semua library berhasil diimport')
```

---

## Load dan Eksplorasi Dataset

Dataset yang digunakan adalah **`vic_electricity`** — data permintaan listrik setengah-jam (*half-hourly*) di Victoria, Australia.

Kolom yang tersedia:
- `Demand` : permintaan listrik (MW) → **target prediksi**
- `Temperature` : suhu rata-rata di Melbourne → **variabel eksogen**
- `Holiday` : apakah hari libur umum

```code-cell
# Download data
# ==============================================================================
data = fetch_dataset(name='vic_electricity')
print('\n--- 5 data pertama ---')
data.head()
```

```code-cell
# Informasi dataset
# ==============================================================================
print(f'Shape dataset  : {data.shape}')
print(f'Kolom          : {data.columns.tolist()}')
print(f'Index (range)  : {data.index.min()} s/d {data.index.max()}')
print(f'Frekuensi      : {data.index.freq}')
print('\nTipe data:')
print(data.dtypes)
print('\nStatistik deskriptif:')
data[['Demand', 'Temperature']].describe()
```

```code-cell
# Agregasi ke frekuensi harian (dari half-hourly)
# ==============================================================================
data = data.resample('D').agg({
    'Demand': 'sum',
    'Temperature': 'mean'
})

print(f'Shape setelah agregasi harian: {data.shape}')
print('\n5 data pertama:')
print(data.head())
```

```code-cell
# Visualisasi time series
# ==============================================================================
fig, axes = plt.subplots(2, 1, figsize=(12, 6), sharex=True)

axes[0].plot(data.index, data['Demand'], color='steelblue', linewidth=0.8)
axes[0].set_title('Permintaan Listrik Harian (Demand) - Victoria, Australia', fontsize=13)
axes[0].set_ylabel('Demand (MW)')
axes[0].grid(True, alpha=0.3)

axes[1].plot(data.index, data['Temperature'], color='orangered', linewidth=0.8)
axes[1].set_title('Suhu Harian Rata-rata (Temperature) - Melbourne', fontsize=13)
axes[1].set_ylabel('Temperature (°C)')
axes[1].set_xlabel('Tanggal')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

---

## Split Data: Train dan Test

```code-cell
# Split data train dan test
# ==============================================================================
end_train = '2014-12-21'

data_train = data.loc[:end_train]
data_test  = data.loc[end_train:]

print(f'Data train: {data_train.index.min()} s/d {data_train.index.max()} | {len(data_train)} hari')
print(f'Data test : {data_test.index.min()}  s/d {data_test.index.max()} | {len(data_test)} hari')

# Visualisasi split
fig, ax = plt.subplots(figsize=(12, 4))
data_train['Demand'].plot(ax=ax, label='Train', color='steelblue')
data_test['Demand'].plot(ax=ax, label='Test', color='orangered')
ax.axvline(pd.Timestamp(end_train), color='black', linestyle='--', label='Batas Train/Test')
ax.set_title('Split Data Train dan Test')
ax.set_ylabel('Demand (MW)')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## Membuat dan Melatih Forecaster

Menggunakan `ForecasterRecursive` dengan:
- **Regressor:** LightGBM (`LGBMRegressor`)
- **Lags:** 7 (menggunakan 7 hari sebelumnya sebagai fitur)
- **Exog:** Temperature (variabel eksogen)

```code-cell
# Membuat dan melatih ForecasterRecursive
# ==============================================================================
forecaster = ForecasterRecursive(
    regressor=LGBMRegressor(
        n_estimators=100,
        learning_rate=0.1,
        random_state=123,
        verbose=-1
    ),
    lags=7  # Gunakan lag_1 sampai lag_7 sebagai fitur input
)

# Fit (latih) model
forecaster.fit(
    y=data_train['Demand'],
    exog=data_train[['Temperature']]
)

print('\n--- Info Forecaster ---')
print(forecaster)
```

---

## Matriks Training (X_train dan y_train)

Di sini kita bisa melihat dengan jelas **bentuk data training** yang digunakan model. `X_train` berisi fitur input (lag_1 sampai lag_7 + Temperature), sedangkan `y_train` berisi target output (nilai Demand yang diprediksi).

```code-cell
# Ekstrak matriks training
# ==============================================================================
X_train, y_train = forecaster.create_train_X_y(
    y=data_train['Demand'],
    exog=data_train[['Temperature']]
)

print('=== BENTUK DATA TRAINING ===')
print(f'X_train (input/fitur)  : {X_train.shape}  → {X_train.shape[0]} baris, {X_train.shape[1]} kolom')
print(f'y_train (output/target): {y_train.shape}  → {y_train.shape[0]} nilai target')

print('\n--- Kolom fitur (X_train) ---')
print(X_train.columns.tolist())

print('\n--- Contoh 5 baris X_train (input) ---')
print(X_train.head())

print('\n--- Contoh 5 baris y_train (output) ---')
print(y_train.head())
```

```code-cell
# Ilustrasi konsep LAG
# ==============================================================================
print('=== ILUSTRASI KONSEP LAG ===')
print('Pada tanggal tertentu, model menggunakan data berikut sebagai INPUT:')
print()

sample_date = X_train.index[10]
print(f'Tanggal prediksi (y)  : {sample_date.date()}')
print(f'lag_1 (kemarin)       : {X_train.loc[sample_date, "lag_1"]:.2f} MW  ← data {(sample_date - pd.Timedelta(days=1)).date()}')
print(f'lag_2 (2 hari lalu)   : {X_train.loc[sample_date, "lag_2"]:.2f} MW  ← data {(sample_date - pd.Timedelta(days=2)).date()}')
print(f'lag_3 (3 hari lalu)   : {X_train.loc[sample_date, "lag_3"]:.2f} MW  ← data {(sample_date - pd.Timedelta(days=3)).date()}')
print(f'lag_7 (7 hari lalu)   : {X_train.loc[sample_date, "lag_7"]:.2f} MW  ← data {(sample_date - pd.Timedelta(days=7)).date()}')
print(f'Temperature (eksogen) : {X_train.loc[sample_date, "Temperature"]:.2f} °C')
print(f'\nOUTPUT / TARGET (y)   : {y_train.loc[sample_date]:.2f} MW  ← ini yang diprediksi')
```

---

## 1. Feature Importance (Bawaan Model LightGBM)

Metode ini langsung dari model LightGBM. Menunjukkan seberapa sering setiap fitur digunakan dalam *split* pohon keputusan saat training.

```code-cell
# Feature Importance dari model internal
# ==============================================================================
feat_importance = forecaster.get_feature_importances()

print('--- Feature Importance ---')
print(feat_importance.to_string(index=False))

# Visualisasi
fig, ax = plt.subplots(figsize=(8, 5))
colors = ['#2196F3' if 'lag' in f else '#FF9800' for f in feat_importance['feature']]
ax.barh(feat_importance['feature'], feat_importance['importance'], color=colors)
ax.set_xlabel('Importance (split count)')
ax.set_title('Feature Importance - LightGBM\n(biru = lag, oranye = eksogen)', fontsize=12)
ax.invert_yaxis()
ax.grid(True, alpha=0.3, axis='x')
plt.tight_layout()
plt.show()
```

---

## 2. Permutation Importance

Mengukur kepentingan fitur dengan mengacak (*shuffle*) nilai suatu fitur, lalu melihat seberapa besar performa model menurun. Fitur yang penting akan menyebabkan penurunan besar jika diacak.

```code-cell
# Permutation Importance
# ==============================================================================
result = permutation_importance(
    estimator=forecaster.estimator,
    X=X_train,
    y=y_train,
    n_repeats=5,
    scoring='neg_mean_squared_error',
    random_state=123
)

perm_df = pd.DataFrame({
    'feature': X_train.columns,
    'importance_mean': result.importances_mean,
    'importance_std': result.importances_std
}).sort_values('importance_mean', ascending=False)

print('--- Permutation Importance ---')
print(perm_df.to_string(index=False))

# Visualisasi
fig, ax = plt.subplots(figsize=(8, 5))
colors = ['#2196F3' if 'lag' in f else '#FF9800' for f in perm_df['feature']]
ax.barh(
    perm_df['feature'],
    perm_df['importance_mean'],
    xerr=perm_df['importance_std'],
    color=colors,
    alpha=0.85,
    capsize=4
)
ax.set_xlabel('Mean Decrease in Score (MSE)')
ax.set_title('Permutation Importance\n(biru = lag, oranye = eksogen)', fontsize=12)
ax.invert_yaxis()
ax.grid(True, alpha=0.3, axis='x')
plt.tight_layout()
plt.show()
```

---

## 3. SHAP Values (SHapley Additive exPlanations)

SHAP adalah metode paling populer untuk explainability model ML. SHAP memberikan dua jenis interpretasi:

- **Global Interpretability**: fitur mana yang paling penting secara keseluruhan
- **Local Interpretability**: kenapa model menghasilkan prediksi tertentu untuk satu observasi

Dibutuhkan dua komponen utama:
1. Internal estimator dari forecaster (`forecaster.estimator`)
2. Matriks training `X_train`

### Membuat SHAP Explainer

```code-cell
# Buat SHAP Explainer
# ==============================================================================
explainer = shap.TreeExplainer(forecaster.estimator)
shap_values = explainer.shap_values(X_train)

print(f'Shape SHAP values: {shap_values.shape}')
print(f'(baris = {shap_values.shape[0]} observasi training, kolom = {shap_values.shape[1]} fitur)')
```

### SHAP Summary Plot — Bar (Global)

```code-cell
# SHAP Summary Plot - Bar (Global Importance)
# ==============================================================================
print('=== SHAP Global Feature Importance (Bar Plot) ===')
shap.summary_plot(
    shap_values,
    X_train,
    plot_type='bar',
    show=True,
    plot_size=(10, 5)
)
```

### SHAP Summary Plot — Beeswarm

Merah = nilai fitur tinggi, Biru = nilai fitur rendah. Posisi kanan = mendorong prediksi naik, Kiri = mendorong prediksi turun.

```code-cell
# SHAP Summary Plot - Beeswarm
# ==============================================================================
shap.summary_plot(
    shap_values,
    X_train,
    show=True,
    plot_size=(10, 5)
)
```

### SHAP Force Plot — Local (satu prediksi)

```code-cell
# SHAP Force Plot - Local (satu prediksi)
# ==============================================================================
print('=== SHAP Force Plot (Local Explainability - prediksi pertama) ===')
print(f'Prediksi untuk tanggal: {X_train.index[0].date()}')
print(f'Nilai prediksi model  : {forecaster.estimator.predict(X_train.iloc[[0]])[0]:.2f} MW')
print(f'Nilai aktual          : {y_train.iloc[0]:.2f} MW\n')

shap.force_plot(
    explainer.expected_value,
    shap_values[0, :],
    X_train.iloc[0, :],
    matplotlib=True,
    figsize=(16, 4)
)
plt.tight_layout()
plt.show()
```

### SHAP Waterfall Plot

```code-cell
# SHAP Waterfall Plot
# ==============================================================================
shap_explanation = shap.Explanation(
    values=shap_values[0],
    base_values=explainer.expected_value,
    data=X_train.iloc[0].values,
    feature_names=X_train.columns.tolist()
)
shap.waterfall_plot(shap_explanation, show=True)
```

---

## 4. Partial Dependence Plots (PDP)

PDP menunjukkan hubungan rata-rata antara satu fitur dan target prediksi, dengan fitur lain dipegang konstan. Berguna untuk memahami pola non-linear antara satu fitur dan output model.

```code-cell
# Partial Dependence Plot (PDP)
# ==============================================================================
features_to_plot = ['lag_1', 'lag_2', 'lag_7', 'Temperature']

fig, axes = plt.subplots(2, 2, figsize=(12, 8))

PartialDependenceDisplay.from_estimator(
    estimator=forecaster.estimator,
    X=X_train,
    features=features_to_plot,
    ax=axes.flatten(),
    grid_resolution=50
)

for ax, feat in zip(axes.flatten(), features_to_plot):
    ax.set_title(f'PDP: {feat}', fontsize=11)
    ax.set_ylabel('Partial Dependence')
    ax.grid(True, alpha=0.3)

plt.suptitle('Partial Dependence Plots\n(hubungan tiap fitur dengan prediksi Demand)',
             fontsize=13, y=1.01)
plt.tight_layout()
plt.show()
```

---

## Bonus: Prediksi ke Depan

```code-cell
# Prediksi 14 hari ke depan
# ==============================================================================
predictions = forecaster.predict(
    steps=14,
    exog=data_test[['Temperature']].iloc[:14]
)

print('Prediksi 14 hari ke depan (MW):')
print(predictions)

# Visualisasi prediksi vs aktual
fig, ax = plt.subplots(figsize=(12, 5))

data_train['Demand'].iloc[-30:].plot(ax=ax, label='Data Aktual (Train)', color='steelblue')
data_test['Demand'].iloc[:14].plot(ax=ax, label='Data Aktual (Test)', color='green', linestyle='--')
predictions.plot(ax=ax, label='Prediksi', color='orangered', marker='o', markersize=4)

ax.set_title('Prediksi vs Aktual Permintaan Listrik\n(14 hari ke depan)', fontsize=12)
ax.set_ylabel('Demand (MW)')
ax.set_xlabel('Tanggal')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

---

## Ringkasan Analisis

| Aspek | Detail |
|---|---|
| **Dataset** | `vic_electricity` — permintaan listrik harian Victoria, Australia |
| **Target** | `Demand` (MW) — total konsumsi listrik per hari |
| **Model** | LightGBM via `ForecasterRecursive` |
| **Fitur input** | `lag_1` s/d `lag_7` (7 hari sebelumnya) + `Temperature` |
| **Lag** | Nilai historis time series yang dijadikan fitur supervised learning |
| **Explainability** | Feature Importance → Permutation → SHAP → PDP |

**Temuan utama dari analisis:**
- `lag_1` (nilai kemarin) adalah fitur paling penting — konsumsi listrik kemarin sangat menentukan konsumsi hari ini
- `Temperature` juga berkontribusi signifikan — suhu mempengaruhi penggunaan AC dan pemanas
- Lag yang lebih jauh (`lag_5`, `lag_6`, `lag_7`) umumnya memiliki kepentingan lebih rendah
- SHAP values memberikan penjelasan yang kaya, baik secara global maupun untuk setiap prediksi individual

---

*Referensi: [Skforecast Docs v0.15.1 — Model Explainability](https://skforecast.org/0.15.1/user_guides/explainability.html)*