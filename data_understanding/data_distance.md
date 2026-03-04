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

# Disctance Measurement 

Distance Measurment atau Mengukur jarak ialah, Teknik matematis yg digunakan untuk menghitung tingkat kesamaan(Similiarity) atau ketidaksamaan (Dissimilarity) antara 2 objek atau titik data.

## Similiarity
Similarity atau Kemiripan adalah ukuran yang menunjukkan seberapa mirip dua objek. Semakin besar nilainya → semakin mirip.
## Dissmilarity
Dissimilarity adalah ukuran yang menunjukkan seberapa berbeda dua objek, Dissimilarity sering disebut sebagai distance (jarak).dan sebaliknya semakin besar nilainya semakin berbeda 2 objek tsbt. Seperti yang kita tahu bahwa dataset memiliki tipe tipe data ada Nominal,Binariy, dan Ordinal, untuk mengukur jarak dari tipe data tersebut memiliki rumus atau hitungan matematis tersendiri. Dibawah ini ada bebrapa methode untuk menghitung jarak untuk tipe data.

## Manhattan 

Manhattan Distance adalah metode untuk menghitung jarak antara dua titik dengan menjumlahkan selisih absolut tiap dimensi. Manhattan Distance dirumuskan sebagai berikut :

$$ d(\mathbf{x}, \mathbf{y}) = \sum_{i=1}^{n} |x_i - y_i| $$ 

Dimana

- x merupakan data pertama
- y merupakan data kedua
- xi merupakan fitur ke-i dari data pertama
- yi merupakan fitur ke-i dari data kedua
## Ecludian

Euclidean distance adalah ukuran jarak “lurus” atau jarak garis lurus antara dua titik dalam ruang n-dimensi. Ini adalah generalisasi jarak Pythagoras ke dimensi lebih tinggi. Euclidean Distance dirumuskan sebagai berikut:

$$ d(\mathbf{p}, \mathbf{q}) = \sqrt{\sum_{i=1}^{n} (p_i - q_i)^2} $$ 

Dimana

- p merupakan data pertama
- q merupakan data kedua
- pi merupakan fitur ke-i dari data pertama
- qi merupakan fitur ke-i dari data kedua

## Biner Simetris dan Asimetris

Biner simetris adalah atribut biner di mana kedua nilai (0 dan 1) sama pentingnya. 
Nilai 1 tidak lebih penting dari 0, begitu pula sebaliknya.

Rumus menghitung jarak data dengan atribut biner simetris adalah:


$$
D = \frac{r + s}{q + r + s + t}
$$

Dimana:
- $\mathbf{q}$ adalah jumlah atribut (1,1) 
- $\mathbf{r}$ adalah jumlah atribut (1,0) 
- $\mathbf{s}$ adalah jumlah atribut (0,1) 
- $\mathbf{t}$ adalah jumlah atribut (0,0) 

|      -       |$\mathbf{Y}=1$|$\mathbf{Y}=0$|
|:------------:|:------------:|:------------:|
|$\mathbf{X}=1$| $\mathbf{q}$ | $\mathbf{r}$ |
|$\mathbf{X}=0$| $\mathbf{s}$ | $\mathbf{t}$ |

### Biner Asimetris
Biner asimetris adalah atribut biner di mana nilai 1 lebih penting daripada nilai 0. Rumus menghitung jarak data dengan atribut biner asimetris sebagai berikut:

$$
D = \frac{r + s}{q + r + s }
$$

## Jarak Atribut Kategorikal
Data kategorikal adalah data berbentuk kategori/label (misalnya: warna, jurusan, status). menghitung jarak data dengan atribut kategorikal sebagai berikut:

$$
D = \frac{u}{p}
$$
Dimana:
- $\mathbf{u}$ adalah jumlah atribut yang berbeda
- $\mathbf{p}$ adalah jumlah sumua atribut (kategorikal)


Lalu juga tak lupa untuk tiap data memiliki perhitungannya sendiri pada markdown dibawah



