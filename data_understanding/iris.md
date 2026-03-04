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

# IRIS 

```{code-cell} 
import pandas as pd
import numpy as np
df = pd.read_csv("../IRIS.csv")
df.head(5)
```

```{code-cell} 
import pandas as pd
import numpy as np

kolom = ['sepal_length', 'sepal_width', 'petal_length', 'petal_width', 'species']
df = pd.read_csv('../IRIS.csv', names=kolom)

kolom_numerik = ['sepal_length', 'sepal_width', 'petal_length', 'petal_width']
kolom_kategori = ['species']

ranges = df[kolom_numerik].max() - df[kolom_numerik].min()

def gower_distance(baris1, baris2, df_ranges):
    total_jarak = 0
    jumlah_kolom = len(kolom_numerik) + len(kolom_kategori)
    
    # Hitung jarak untuk kolom Numerik (Angka)
    for col in kolom_numerik:
        # Rumus: |x1 - x2| / Range
        jarak_num = abs(baris1[col] - baris2[col]) / df_ranges[col]
        total_jarak += jarak_num
        
    # Hitung jarak untuk kolom Kategorik (Teks/Spesies)
    for col in kolom_kategori:
        # Rumus: 0 jika sama, 1 jika beda
        jarak_kat = 0 if baris1[col] == baris2[col] else 1
        total_jarak += jarak_kat
        
    # Rata-ratakan jarak dari seluruh kolom
    gower_dist = total_jarak / jumlah_kolom
    return gower_dist

bunga_1 = df.iloc[0]

bunga_2 = df.iloc[50]

print("=== DATA BUNGA 1 ===")
print(bunga_1.to_frame().T)
print("\n=== DATA BUNGA 2 ===")
print(bunga_2.to_frame().T)

# Memanggil fungsi perhitungan
hasil_jarak = gower_distance(bunga_1, bunga_2, ranges)

print(f"\n=> Jarak Campuran (Gower's Distance) antara Bunga 1 dan Bunga 2 adalah: {hasil_jarak:.4f}")
```