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

## Linear Regression
## 1. Pendahuluan

Regresi linear sederhana adalah metode statistik yang digunakan untuk memodelkan hubungan linier antara satu variabel independen (X) dan satu variabel dependen (Y). Tujuan utamanya adalah menemukan garis lurus terbaik yang merepresentasikan pola hubungan tersebut dalam data.

Model regresi linear dinyatakan sebagai:

```
Ŷ = β̂₀ + β̂₁X
```

Di mana:
- `Ŷ` = nilai Y yang diprediksi
- `β̂₀` = intercept (titik potong sumbu Y)
- `β̂₁` = koefisien regresi (kemiringan garis)

---

## 2. Data Penelitian

Berikut adalah data yang digunakan dalam analisis:

| No | X | Y |
|----|---|---|
| 1  | 2 | 2 |
| 2  | 4 | 3 |
| 3  | 3 | 5 |
| 4  | 3 | 4 |
| 5  | 3 | 3 |
| 6  | 4 | 5 |
| 7  | 5 | 6 |

**Jumlah observasi (n):** 7

---

## 3. Landasan Teori

### 3.1 Estimasi Parameter dengan OLS

Parameter regresi β̂ diperoleh menggunakan metode **Ordinary Least Squares (OLS)**, yaitu dengan meminimalkan jumlah kuadrat error (SSE).

Rumus estimasi parameter dalam bentuk matriks:

$$\hat{\beta} = (X^T X)^{-1} X^T Y$$

Di mana:
- `X` = matriks desain (dengan kolom ones untuk intercept)
- `Y` = vektor variabel dependen
- `X^T` = transpose dari matriks X
- `(X^T X)^{-1}` = invers dari perkalian matriks X^T dan X

### 3.2 Matriks Desain

Matriks desain X dibentuk dengan menambahkan kolom konstanta 1 (untuk intercept):

```
X = [1  x₁]   Y = [y₁]
    [1  x₂]       [y₂]
    [1  x₃]       [y₃]
    [...]          [...]
```

---

## 4. Perhitungan Manual (OLS)

### 4.1 Membangun Matriks X dan Y

```
X = [[1, 2],    Y = [[2],
     [1, 4],         [3],
     [1, 3],         [5],
     [1, 3],         [4],
     [1, 3],         [3],
     [1, 4],         [5],
     [1, 5]]         [6]]
```

### 4.2 Hitung X^T X

```
X^T X = [[n,    ΣXᵢ  ],
         [ΣXᵢ,  ΣXᵢ²]]

n    = 7
ΣX   = 2+4+3+3+3+4+5 = 24
ΣX²  = 4+16+9+9+9+16+25 = 88

X^T X = [[7,  24],
         [24, 88]]
```

### 4.3 Hitung X^T Y

```
X^T Y = [[ΣYᵢ  ],
         [ΣXᵢYᵢ]]

ΣY    = 2+3+5+4+3+5+6 = 28
ΣXY   = (2×2)+(4×3)+(3×5)+(3×4)+(3×3)+(4×5)+(5×6)
      = 4+12+15+12+9+20+30 = 102

X^T Y = [[28 ],
         [102]]
```

### 4.4 Hitung Invers (X^T X)⁻¹

Untuk matriks 2×2: `[[a,b],[c,d]]`, inversnya = `1/(ad-bc) × [[d,-b],[-c,a]]`

```
det(X^T X) = (7 × 88) - (24 × 24) = 616 - 576 = 40

(X^T X)⁻¹ = 1/40 × [[88, -24],
                      [-24,  7]]

           = [[2.2,  -0.6],
              [-0.6,  0.175]]
```

### 4.5 Hitung β̂ = (X^T X)⁻¹ X^T Y

```
β̂₀ = (2.2 × 28) + (-0.6 × 102)  = 61.6 - 61.2  = 0.40
β̂₁ = (-0.6 × 28) + (0.175 × 102) = -16.8 + 17.85 = 1.05
```

---

## 5. Hasil Analisis

### 5.1 Persamaan Regresi

```
Ŷ = 0.40 + 1.05X
```

| Parameter | Nilai |
|-----------|-------|
| Intercept (β̂₀) | 0.40 |
| Koefisien (β̂₁) | 1.05 |

### 5.2 Nilai Fitted dan Residual

| No | X | Y (Aktual) | Ŷ (Prediksi) | Residual (Y - Ŷ) |
|----|---|------------|--------------|-------------------|
| 1  | 2 | 2          | 2.50         | -0.50             |
| 2  | 4 | 3          | 4.60         | -1.60             |
| 3  | 3 | 5          | 3.55         | +1.45             |
| 4  | 3 | 4          | 3.55         | +0.45             |
| 5  | 3 | 3          | 3.55         | -0.55             |
| 6  | 4 | 5          | 4.60         | +0.40             |
| 7  | 5 | 6          | 5.65         | +0.35             |

### 5.3 Evaluasi Model

```
Ȳ  = 28/7 = 4.00

SST (Total SS)     = Σ(Yᵢ - Ȳ)²   = 12.00
SSE (Residual SS)  = Σ(Yᵢ - Ŷᵢ)²  ≈ 5.48
SSR (Regression SS)= SST - SSE     ≈ 6.52

R² = SSR / SST ≈ 0.54
```

| Metrik | Nilai |
|--------|-------|
| R² (Koefisien Determinasi) | ~0.54 |
| Interpretasi | Model menjelaskan ~54% variansi Y |

---

## 6. Implementasi Python

### 6.1 Metode OLS Manual (NumPy)

```python
import numpy as np

X_raw = np.array([2, 4, 3, 3, 3, 4, 5])
Y     = np.array([2, 3, 5, 4, 3, 5, 6])

# Matriks desain dengan kolom intercept
ones = np.ones(len(X_raw))
X    = np.column_stack([ones, X_raw])

# Rumus: β̂ = (X^T X)^{-1} X^T Y
XtX     = X.T @ X
XtY     = X.T @ Y
beta    = np.linalg.inv(XtX) @ XtY

print(f"β̂₀ (Intercept) : {beta[0]:.4f}")
print(f"β̂₁ (Koefisien) : {beta[1]:.4f}")
```

### 6.2 Metode scikit-learn

```python
from sklearn.linear_model import LinearRegression

X = np.array([2, 4, 3, 3, 3, 4, 5]).reshape(-1, 1)
Y = np.array([2, 3, 5, 4, 3, 5, 6])

model = LinearRegression()
model.fit(X, Y)

print(f"β̂₀ : {model.intercept_:.4f}")
print(f"β̂₁ : {model.coef_[0]:.4f}")
print(f"R²  : {model.score(X, Y):.4f}")
```

---

## 7. Interpretasi Hasil

1. **Intercept (β̂₀ = 0.40):** Ketika nilai X = 0, prediksi Y adalah 0.40. Dalam konteks praktis, nilai ini merupakan konstanta dasar model.

2. **Koefisien (β̂₁ = 1.05):** Setiap kenaikan 1 satuan pada X akan meningkatkan Y sebesar 1.05 satuan. Hubungan X dan Y bersifat **positif**.

3. **Koefisien Determinasi (R² ≈ 0.54):** Model regresi mampu menjelaskan sekitar **54%** dari total variasi dalam Y. Nilai ini menunjukkan hubungan positif yang cukup kuat, namun masih ada variasi yang belum dijelaskan oleh model.

---

## 8. Kesimpulan

Analisis regresi linear sederhana menggunakan metode OLS (dengan rumus $\hat{\beta} = (X^T X)^{-1} X^T Y$) menghasilkan persamaan:

**Ŷ = 0.40 + 1.05X**

Terdapat hubungan linear positif antara variabel X dan Y. Model ini dapat digunakan untuk memprediksi nilai Y berdasarkan nilai X, dengan tingkat akurasi yang memadai (R² ≈ 54%).

---

## Referensi

- Montgomery, D. C., & Runger, G. C. (2014). *Applied Statistics and Probability for Engineers*. Wiley.
- James, G., et al. (2021). *An Introduction to Statistical Learning*. Springer.
- scikit-learn Documentation: [https://scikit-learn.org](https://scikit-learn.org)