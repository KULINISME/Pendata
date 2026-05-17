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
# Random Forest
## Pengertian
random forest adalah algoritma yang menggabungkan hasil (output) dari beberapa decision tree untuk mencapai satu hasil yang lebih akurat. Random forest membutuhkan gabungan beberapa decision tree untuk memprediksi hasil yang akurat. 

Konsep sederhana dari random forest adalah beberapa decision tree yang tidak berkorelasi akan bekerja lebih baik sebagai kelompok dibandingkan individu. 

Saat menggunakan random forest sebagai pengklasifikasi, satu decision tree menyumbang satu suara. Setiap decision tree bisa menghasilkan jawaban yang sama atau berbeda satu sama lain. Misalnya decision tree A, B, E dan F memprediksi hasil 1. Sementara decision tree C dan D memprediksi hasil 0.

Karena ada banyaknya alternatif jawaban dalam decision tree dan kemungkinan bias yang tinggi, random forest mengambil prediksi hasil dari beberapa decision tree berdasarkan suara mayoritas dan memprediksi hasil yang lebih akur