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

# Ordinal

Atribut Ordinal <br>
Atribut ordinal adalah atribut yang:
- Memiliki urutan / tingkatan
- Tetapi selisih antar tingkat tidak selalu sama
- Tidak bisa langsung diperlakukan sebagai numerik murni

Pada dataset saya terdapat 2 kolom tipe data Ordinal,diet_quality dan internet_quality

```{code-cell} 
import pandas as pd

# 1. Load dataset
df = pd.read_csv("../student_habits_performance.csv")

# 2. Ambil dua sampel data untuk dibandingkan (contoh: Index 0 dan Index 2)
# Index 0: internet_quality = 'Average'
# Index 2: internet_quality = 'Poor'
row1 = df.iloc[0]
row2 = df.iloc[2]

# 3. Definisikan Mapping Ordinal (Poor < Average < Good)
# Kita petakan ke rentang 0 sampai 1
internet_map = {
    'Poor': 0.0,
    'Average': 0.5,
    'Good': 1.0
}

# 4. Konversi nilai tekstual ke nilai numerik ordinal
val1 = internet_map[row1['internet_quality']]
val2 = internet_map[row2['internet_quality']]

# 5. Hitung Jarak Absolut
dist_internet = abs(val1 - val2)

print(f"Data 1 ({row1['internet_quality']}): {val1}")
print(f"Data 2 ({row2['internet_quality']}): {val2}")
print("-" * 30)
print(f"Jarak Internet Quality: {dist_internet:.1f}")
```
Pada tipe data ordinal saya transformasikan terlebih dahulu dari kolom internet_quality dan diet_quality diberi nilai untuk tiap inputnya Poor = 1, Fair = 2, dan Good = 3, lalu ketika sudah didapatkan bisa dihitung nilainya secara Ecludian dan Manhattan