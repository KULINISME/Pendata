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

# Binary
Atribut Binary <br>
Atribut binary adalah atribut yang hanya memiliki dua kemungkinan nilai,ciri-ciri atribut Binary:
- Hanya Memiliki 2 Nilai
- dikodekan 1 atau 0
- Tidak memiliki nilai tengah

```{code-cell} 
import pandas as pd
import numpy as np
df = pd.read_csv("../student_habits_performance.csv")
df.head(5)
```

```{code-cell} 
import pandas as pd
import numpy as np

# Load dataset
df = pd.read_csv("../student_habits_performance.csv")

# Ambil dua mahasiswa
s1000 = df[df['student_id'] == 'S1000']
s1001 = df[df['student_id'] == 'S1001']

# Tentukan kolom binary
binary_cols = ['part_time_job']  # tambahkan jika ada

# Konversi Yes/No ke 1/0
mapping = {'Yes': 1, 'No': 0}

x = s1000[binary_cols].replace(mapping).astype(int).values[0]
y = s1001[binary_cols].replace(mapping).astype(int).values[0]

# Hitung a, b, c
a = np.sum((x == 1) & (y == 1))
b = np.sum((x == 1) & (y == 0))
c = np.sum((x == 0) & (y == 1))

# Cegah pembagian nol
if (a + b + c) == 0:
    jaccard_distance = 0
else:
    jaccard_distance = (b + c) / (a + b + c)

print("a (1,1):", a)
print("b (1,0):", b)
print("c (0,1):", c)
print("Jaccard Distance:", jaccard_distance)
```