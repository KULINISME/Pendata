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

Pada kali ini akan menghitung jarak antara data 1 dan 2 pada datset IRIS 

```{code-cell} 
import pandas as pd
import numpy as np
df = pd.read_csv("../IRIS.csv")
df.head(5)
```
Saya menggunakan metode Ecludian:

$$
\begin{aligned}
d(\mathbf{p}, \mathbf{q}) &= \frac{1}{n} \sum_{i=1}^{n} \delta_i(p_i, q_i) \\[10pt]
d(\mathbf{1}, \mathbf{2}) &= \frac{1}{5} \left( \frac{|5.1 - 4.9|}{3.6} + \frac{|3.5 - 3.0|}{2.4} + \frac{|1.4 - 1.4|}{5.9} + \frac{|0.2 - 0.2|}{2.4} + 0 \right) \\[10pt]
d(\mathbf{1}, \mathbf{2}) &= \frac{1}{5} \left( \frac{0.2}{3.6} + \frac{0.5}{2.4} + 0 + 0 + 0 \right) \\[10pt]
d(\mathbf{1}, \mathbf{2}) &= \frac{1}{5} (0.0555 + 0.2083 + 0 + 0 + 0) \\[10pt]
d(\mathbf{1}, \mathbf{2}) &= \frac{0.2638}{5} = 0.05276
\end{aligned}
$$

```{figure} ../image/ecludian_iris.PNG
---
width: 60%
align: center
---
IRIS Metode Eckudian
```

```{code-cell} 
import pandas as pd
import numpy as np

# 1. Load dataset
df = pd.read_csv("../IRIS.csv")

# 2. Ambil Data 1 (index 0) dan Data 2 (index 1)
row1 = df.iloc[0]
row2 = df.iloc[1]

print("--- Data 1 ---")
print(row1.values)
print("\n--- Data 2 ---")
print(row2.values)
print("-" * 30)

# Menentukan kolom numerik
num_cols = ['sepal_length', 'sepal_width', 'petal_length', 'petal_width']

# 3. Hitung Range (Maksimum - Minimum) dari seluruh data untuk fitur numerik
ranges = df[num_cols].max() - df[num_cols].min()

# List untuk menyimpan jarak masing-masing atribut
gower_dists = []

# 4. Hitung Jarak Atribut Numerik: |x1 - x2| / range
for col in num_cols:
    distance = abs(row1[col] - row2[col]) / ranges[col]
    gower_dists.append(distance)
    print(f"Jarak {col}: {distance:.4f}")

# 5. Hitung Jarak Atribut Kategorikal: 0 jika sama, 1 jika beda
if row1['species'] == row2['species']:
    cat_dist = 0
else:
    cat_dist = 1

gower_dists.append(cat_dist)
print(f"Jarak species: {cat_dist}")

# 6. Hitung Total Jarak Gower (Rata-rata dari semua jarak atribut)
gower_distance = np.mean(gower_dists)

print("-" * 30)
print(f"Total Jarak Gower (Data 1 & Data 2): {gower_distance:.5f}")
```