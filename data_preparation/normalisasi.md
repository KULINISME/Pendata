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

# Normalisasi Data

Data yang dikumpulkan dalam sebuah himpunan data (data set) mungkin belum cukup berguna untuk sebuah algoritma Data Mining (DM). Terkadang, atribut yang dipilih merupakan atribut mentah yang memiliki makna di domain asalnya, atau dirancang khusus untuk bekerja pada sistem operasional tempat data tersebut saat ini digunakan. Biasanya, atribut-atribut asli ini tidak cukup baik untuk menghasilkan model prediktif yang akurat. Oleh karena itu, adalah hal yang umum untuk melakukan serangkaian langkah manipulasi guna mentransformasi atribut asli atau untuk menghasilkan atribut baru dengan sifat yang lebih baik, yang nantinya akan meningkatkan kekuatan prediktif dari model tersebut. Atribut baru ini biasanya disebut sebagai variabel pemodelan (modeling variables) atau variabel analitik (analytic variables).

Pada bagian ini, kita akan berfokus pada transformasi yang tidak menghasilkan atribut baru, melainkan mengubah distribusi dari nilai aslinya menjadi sekumpulan nilai baru dengan sifat-sifat yang diinginkan. Lalu ada 3 metode untuk melakukan normalisasi:
1. MinMax
2. Z-Score
3. Decimal Scalling
