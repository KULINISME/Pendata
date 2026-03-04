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

# Nominal

 Atribut Nominal <br>
Atribut nominal adalah tipe data kategorikal yang:
- Tidak memiliki urutan / ranking
- Hanya berfungsi sebagai label atau nama kategori
- Tidak bisa dibandingkan lebih besar / lebih kecil

Lalu untuk contoh,sebagai berikut:
Warna: Merah,Kuning,Hijau
Jenis Kelamin: Pria dan Wanita

```{code-cell} 
import pandas as pd
import numpy as np
df = pd.read_csv("../student_habits_performance.csv")
df.head(5)
```

Untuk dataset diakse sebagai berikut:[Kaggle Student Habits Dataset](https://www.kaggle.com/code/jayaantanaath/student-habits-vs-academic-performance-ml-90/notebook)


```{code-cell} 
df_numeric = df.select_dtypes(include=[np.number])
print(df_numeric.dtypes)
```