---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: code-cell 3
  language: code-cell
  name: code-cell3
---

# Dokumentasi Notebook: Prediksi Kadar NO2 di Daerah Jombang
Menggunakan K-Nearest Neighbors (KNN) Regression

---

## 1. Pendahuluan

Notebook ini mendokumentasikan pipeline lengkap untuk pengambilan data, preprocessing, dan pemodelan prediksi kadar Nitrogen Dioksida (NO2) di wilayah Jombang menggunakan algoritma K-Nearest Neighbors (KNN) Regression. Data NO2 diperoleh dari Sentinel-5P melalui platform OpenEO.

---

## 2. Pengambilan Data

### 2.1 Koneksi ke OpenEO

Koneksi ke server OpenEO dilakukan menggunakan library `openeo` dengan autentikasi OIDC.

```code-cell
:tags: [skip-execution]
import openeo
connection = openeo.connect("openeo.dataspace.copernicus.eu").authenticate_oidc()
```

### 2.2 Definisi Area of Interest (AOI)

Area penelitian didefinisikan menggunakan polygon GeoJSON yang mencakup wilayah Jombang, Jawa Timur.

| Parameter | Nilai |
|-----------|-------|
| Batas Barat (West) | 112.17337695661712 |
| Batas Selatan (South) | -7.608680889617773 |
| Batas Timur (East) | 112.29947621158374 |
| Batas Utara (North) | -7.515673004000362 |

### 2.3 Load Data Sentinel-5python NO2

Data NO2 dari koleksi `SENTINEL_5P_L2` diambil untuk rentang waktu 2023-10-01 hingga 2025-10-01 dengan band NO2.

```code-cell
:tags: [skip-execution]
s5post = connection.load_collection(
    "SENTINEL_5P_L2",
    temporal_extent=["2023-10-01", "2025-10-01"],
    spatial_extent={
        "west": 112.17337695661712,
        "south": -7.608680889617773,
        "east": 112.29947621158374,
        "north": -7.515673004000362
    },
    bands=["NO2"],
)
```

### 2.4 Agregasi Temporal dan Spasial

Data diagregasi secara temporal (harian) untuk menghindari duplikasi, kemudian diagregasi secara spasial menggunakan mean agar menghasilkan satu nilai per hari untuk seluruh area.

```code-cell
:tags: [skip-execution]
# Agregasi harian
s5p_no2_daily = s5post.aggregate_temporal_period(reducer="mean", period="day")

# Agregasi spasial
s5p_no2_aoi = s5p_no2_daily.aggregate_spatial(reducer="mean", geometries=aoi)
```

### 2.5 Eksekusi Batch Job

Job dieksekusi secara batch dan hasilnya disimpan dalam format NetCDF.

```code-cell
:tags: [skip-execution]
job = s5post.execute_batch(title="NO2 in Jombang", outputfile="NO2Jombang.nc")
```

---

## 3. Preprocessing Data

### 3.1 Membaca File NetCDF

File hasil download dibaca menggunakan library `netCDF4`. Variabel yang diambil adalah NO2 dan waktu (t).

```code-cell
:tags: [skip-execution]
import netCDF4

ds = netCDF4.Dataset("NO2Jombang.nc")
no2 = ds.variables["NO2"][:]
time = ds.variables["t"][:]
dates = netCDF4.num2date(time, units=time_units)
```

Struktur data NO2 yang dihasilkan:

| Deskripsi | Nilai |
|-----------|-------|
| Tipe data | `numpy.ma.core.MaskedArray` |
| Jumlah record | 725 |
| Ukuran grid (baris) | 9 |
| Ukuran grid (kolom) | 8 |
| Contoh nilai | `3.7701793e-05` |

### 3.2 Interpolasi Missing Values pada Grid

Data NO2 dalam grid 9×8 diinterpolasi secara linear untuk mengisi nilai yang hilang (masked values) pada setiap titik grid.

```code-cell
:tags: [skip-execution]
import numpy as np
import pandas as pd

no2_filled = np.zeros_like(no2).filled(0)

for i in range(no2.shape[1]):     # 9 baris
    for j in range(no2.shape[2]): # 8 kolom
        series = pd.Series(no2[:, i, j])
        no2_filled[:, i, j] = series.interpolate(method='linear', limit_direction='both').to_numpy()
```

### 3.3 Konversi ke Format Time Series

Data dikonversi menjadi format time series (tanggal dan nilai rata-rata NO2) lalu disimpan ke file CSV.

```code-cell
:tags: [skip-execution]
new_dates, new_no2 = [], []
for i in range(len(dates)):
    new_date = dates[i].strftime('%Y-%m-%d')
    new_dates.append(new_date)
    new_no2.append(np.mean(no2_filled[i]))

df = pd.DataFrame({"date": new_dates, "NO2": new_no2})
df.to_csv("NO2_Jombang_timeseries.csv", index=False)
```

### 3.4 Pengecekan dan Pengisian Tanggal Hilang

Rentang tanggal lengkap 2023-10-01 hingga 2025-09-30 dibuat, lalu tanggal yang hilang diisi dengan interpolasi linear berbasis waktu.

```code-cell
:tags: [skip-execution]
full_range = pd.date_range(start="2023-10-01", end="2025-09-30", freq='D')
df = df.set_index('date').reindex(full_range)
df['NO2'] = df['NO2'].interpolate(method='time')
df['NO2'] = df['NO2'].fillna(method='bfill').fillna(method='ffill')
```

### 3.5 Deteksi dan Penanganan Outlier (IQR)

Outlier dideteksi menggunakan metode IQR (Interquartile Range). Nilai di luar batas 1.5×IQR ditandai sebagai outlier, lalu diganti dengan interpolasi linear.

```code-cell
:tags: [skip-execution]
Q1 = df['NO2'].quantile(0.25)
Q3 = df['NO2'].quantile(0.75)
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Tandai outlier menjadi NaN
df['NO2_cleaned'] = df['NO2'].mask(
    (df['NO2'] < lower_bound) | (df['NO2'] > upper_bound)
)

# Interpolasi untuk mengisi outlier
df['NO2_filled'] = df['NO2_cleaned'].interpolate(method='linear')
df['NO2_filled'] = df['NO2_filled'].bfill().ffill()
```

### 3.6 Normalisasi Data (Min-Max Scaling)

Data NO2 dinormalisasi ke rentang [0, 1] menggunakan `MinMaxScaler` dari scikit-learn.

```code-cell
:tags: [skip-execution]
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()
df['NO2_scaled'] = scaler.fit_transform(df[['NO2']])
```

---

## 4. Pembentukan Data Supervised

Data time series diubah menjadi format supervised learning dengan membuat fitur lag. Fungsi `create_supervised` membuat kolom `NO2(t-n)` hingga `NO2(t-1)` sebagai fitur, dan `NO2(t)` sebagai label.

```code-cell
:tags: [skip-execution]
def create_supervised(data, n_lag=4):
    df_supervised = pd.DataFrame()

    for i in range(n_lag, 0, -1):
        df_supervised[f'NO2(t-{i})'] = data.shift(i)

    df_supervised['NO2(t)'] = data
    df_supervised.dropna(inplace=True)

    return df_supervised
```

Tiga variasi lag diuji untuk menentukan konfigurasi terbaik:

| Variabel | Nilai Lag | Deskripsi |
|----------|-----------|-----------|
| `supervised_df` | 4 | Menggunakan 4 hari sebelumnya sebagai fitur |
| `supervised_df10` | 10 | Menggunakan 10 hari sebelumnya sebagai fitur |
| `supervised_df50` | 50 | Menggunakan 50 hari sebelumnya sebagai fitur |

Pemilihan nilai lag juga didukung oleh analisis korelasi antara fitur lag dengan target `NO2(t)`.

---

## 5. Pemodelan KNN Regression

### 5.1 Arsitektur Model

Model KNN Regression digunakan dengan parameter `n_neighbors=5`. Data dibagi menjadi 80% training dan 20% testing tanpa shuffle untuk menjaga urutan temporal.

```code-cell
:tags: [skip-execution]
from sklearn.neighbors import KNeighborsRegressor
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, shuffle=False
)

knn = KNeighborsRegressor(n_neighbors=5)
knn.fit(X_train, y_train)
y_pred = knn.predict(X_test)
```

### 5.2 Metrik Evaluasi

Tiga metrik digunakan untuk mengevaluasi performa model:

| Metrik | Formula | Keterangan |
|--------|---------|------------|
| RMSE | `sqrt(mean((y_true - y_pred)²))` | Root Mean Square Error — mengukur rata-rata besar kesalahan |
| R² Score | `1 - SS_res / SS_tot` | Koefisien determinasi — mengukur proporsi variansi yang dijelaskan model |
| MAPE | `mean(|y_true - y_pred| / y_true) * 100` | Mean Absolute Percentage Error — kesalahan dalam satuan persen |

### 5.3 Perbandingan Eksperimen

Tiga model dilatih dengan konfigurasi lag yang berbeda:

```code-cell
:tags: [skip-execution]
knn_4,  y_test_4,  y_pred_4  = train_knn(supervised_df,   "KNN - 4 Hari Sebelumnya")
knn_10, y_test_10, y_pred_10 = train_knn(supervised_df10, "KNN - 10 Hari Sebelumnya")
knn_50, y_test_50, y_pred_50 = train_knn(supervised_df50, "KNN - 50 Hari Sebelumnya")
```

---

## 6. Visualisasi Hasil

Hasil prediksi dari setiap konfigurasi model divisualisasikan dalam bentuk grafik perbandingan antara nilai aktual dan prediksi menggunakan matplotlib.

```code-cell
:tags: [skip-execution]
plt.figure()
plt.plot(np.arange(len(y_test_4)), y_test_4, label="Actual")
plt.plot(np.arange(len(y_pred_4)), y_pred_4, label="Predicted")
plt.title("KNN Regression - 4 Hari Sebelumnya")
plt.xlabel("Sample Index")
plt.ylabel("NO2 Value")
plt.legend()
plt.show()
```

Visualisasi serupa juga dibuat untuk model KNN - 10 Hari Sebelumnya dan KNN - 50 Hari Sebelumnya.

---

## 7. Ringkasan Alur Kerja (Pipeline)

| No. | Tahap | Tools / Library | Output |
|-----|-------|-----------------|--------|
| 1 | Koneksi & Unduh Data | `openeo` | File `NO2Jombang.nc` |
| 2 | Baca File NetCDF | `netCDF4` | Array NO2 (725 × 9 × 8) |
| 3 | Interpolasi Grid | `numpy`, `pandas` | Array NO2 tanpa missing |
| 4 | Konversi ke CSV | `pandas` | `NO2_Jombang_timeseries.csv` |
| 5 | Isi Tanggal Hilang | `pandas` | Time series lengkap |
| 6 | Deteksi & Hapus Outlier | `pandas` | Data NO2 bersih |
| 7 | Normalisasi | `sklearn` MinMaxScaler | Data dalam skala [0, 1] |
| 8 | Buat Data Supervised | `pandas` | DataFrame fitur & label |
| 9 | Latih Model KNN | `sklearn` KNeighborsRegressor | Model terlatih |
| 10 | Evaluasi & Visualisasi | `sklearn` metrics, `matplotlib` | RMSE, R², MAPE, grafik |

---

## 8. Catatan Teknis

- Library utama yang digunakan: `openeo`, `netCDF4`, `numpy`, `pandas`, `scikit-learn`, `matplotlib`.
- Data time series mencakup rentang 2023-10-01 hingga 2025-09-30 (2 tahun penuh).
- Model tidak menggunakan shuffle saat split untuk menjaga urutan temporal data.
- Outlier pada data NO2 ditangani dengan metode IQR dan diisi ulang dengan interpolasi linear.
- Analisis korelasi lag hingga 30 hari dilakukan untuk mendukung pemilihan nilai lag yang optimal.