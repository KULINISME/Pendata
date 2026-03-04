---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Data Description

Langkah berikutnya adalah deskripsi data (data description). Pada tahap ini dilakukan identifikasi terhadap jumlah data, jumlah atribut, tipe data, serta makna dari setiap variabel yang tersedia. Peneliti perlu memahami mana variabel yang berperan sebagai fitur (independen) dan mana yang menjadi target (dependen), jika ada. Tujuan dari langkah ini adalah untuk mengetahui struktur dasar dataset sehingga tidak terjadi kesalahan dalam proses analisis selanjutnya, seperti salah menginterpretasikan tipe data atau fungsi suatu atribut.

# Tujuan Data Description

Berikut adalah tujuan Data Description (Describe Data):
1. Mengetahui struktur dataset, seperti jumlah baris (record) dan kolom (atribut).
2. Mengidentifikasi tipe data pada setiap variabel (numerik, kategorikal, tanggal, dll.).
3. Memahami arti dan fungsi setiap atribut dalam dataset.
4. Menentukan peran variabel, seperti fitur (independen) dan target (dependen) jika ada.
5. Mengetahui gambaran umum data sebelum dilakukan analisis lebih lanjut.
6. Mencegah kesalahan interpretasi data, seperti salah membaca tipe atau makna variabel.
7. Menjadi dasar untuk tahap eksplorasi dan persiapan data selanjutnya.

# Studi Kasus
Pada iris flower datasets didapatkan 4 attribute, sebagai berikut:
1. sepal_length(numerik)
2. sepal_width(numerik)
3. petal_length(numerik)
4. petal_width(numerik)

Lalu adapun class atau label
species(kategori),dan pada class species ini terbagi menjadi 3,yaitu:
1. Iris-setosa 50 Data
2. Iris-versicolor 50 Data
3. Iris-virginica 50 Data


![Grafik Data](../image/datadesc.PNG)

Kita ambil sampel dari salah satu attribute tersebut yaitu sepal_width kita bedah lebih detail
![Grafik Data](../image/description.PNG)

Implementasi dari Python:
```{code-cell} 

import pandas as pd
from scipy import stats
df=pd.read_csv("../IRIS.csv")
kolom = df['sepal_width']

print("Jumlah data        :", kolom.count())
print("Rata-rata          :", round(kolom.mean(), 2))
print("Nilai minimal      :", kolom.min())
print("Q1                 :", kolom.quantile(0.25))
print("Q2 (Median)        :", kolom.quantile(0.5))
print("Q3                 :", kolom.quantile(0.75))
print("Nilai maksimal     :", kolom.max())
print("Kemencengan (skew) :", round(kolom.skew(), 2))
mode = stats.mode(kolom, keepdims=True)
print("Nilai modus        :", mode.mode[0])
print("Jumlah modus       :", mode.count[0])

print("Standar deviasi    :", round(kolom.std(), 2))
print("Variansi           :", round(kolom.var(), 2))
```


```{note} 

Dari data diatas didapatkan informasi mengenai struktur maupun peran dari tiap variabel terkait 1 fitur yg ada di Iris Flower Dataset

```