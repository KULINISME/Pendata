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

```{code-cell} 
import pandas as pd
import numpy as np
df = pd.read_csv("../student_habits_performance.csv")
df.head(5)
```