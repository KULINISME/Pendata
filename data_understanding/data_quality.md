# Data Quality Verification

Tahap selanjutnya adalah verifikasi kualitas data (data quality verification). Pada langkah ini dilakukan pemeriksaan terhadap kemungkinan adanya missing value, data duplikat, outlier, inkonsistensi format, maupun kesalahan input. Tujuan utama dari tahap ini adalah memastikan bahwa data yang akan dianalisis dalam kondisi baik dan layak digunakan. Data yang tidak berkualitas dapat menghasilkan kesimpulan yang bias atau model yang tidak akurat, sehingga tahap ini menjadi bagian yang sangat krusial dalam proses analisis data. Berikut hal hal yg harus diperhatikan pada tahap ini:

## Missing Value
Missing value adalah data yang tidak terisi atau bernilai null pada suatu atribut. Kondisi ini bisa terjadi karena kesalahan input, data tidak tersedia, atau proses pengumpulan data yang tidak lengkap. Jika jumlahnya kecil, missing value bisa dihapus atau diisi (imputasi). Namun, jika jumlahnya besar, dapat memengaruhi akurasi analisis dan perlu penanganan khusus.
## Data Duplikat
Data duplikat adalah baris data yang tercatat lebih dari satu kali dengan isi yang sama. Duplikasi dapat menyebabkan hasil analisis menjadi bias karena suatu data dihitung lebih dari sekali. Pada tahap ini dilakukan pengecekan apakah terdapat record yang identik dan perlu dihapus agar dataset tetap akurat.
## Outlier
Outlier adalah nilai yang menyimpang jauh dari mayoritas data lainnya. Keberadaan outlier dapat memengaruhi perhitungan statistik seperti rata-rata dan standar deviasi, serta berdampak pada model tertentu seperti regresi atau clustering. Dalam tahap ini dilakukan identifikasi apakah outlier tersebut merupakan kesalahan data atau memang representasi kejadian ekstrem yang valid.
## Inkonsistensi Format Data
Inkonsistensi terjadi ketika format penulisan data tidak seragam. Contohnya perbedaan format tanggal (DD/MM/YYYY dan MM/DD/YYYY), atau kategori yang ditulis berbeda seperti “L”, “Laki-laki”, dan “Male”. Ketidakkonsistenan ini dapat menyulitkan analisis karena sistem dapat menganggapnya sebagai kategori yang berbeda. Oleh sebab itu, perlu dilakukan standarisasi format.
## Ketidaksesuain Data
Terkadang tipe data tidak sesuai dengan seharusnya, misalnya angka tersimpan sebagai teks (string). Hal ini dapat menghambat proses perhitungan statistik atau pemodelan. Pada tahap ini dilakukan pengecekan agar setiap atribut memiliki tipe data yang benar sesuai kebutuhan analisis.

# Kesimpulan
```{figure} ../image/Kesimpulan.png
---
width: 60%
align: center
---
Kesimpulan
```
 <br>
Berdasarkan hasil pemeriksaan pada tahap Data Quality Verification, dataset Iris dapat dikatakan sebagai dataset yang berkualitas baik. Hal ini karena tidak ditemukan missing value pada setiap atribut, tidak terdapat data duplikat, serta tidak ditemukan kesalahan input atau nilai yang tidak logis. Selain itu, tipe data pada setiap variabel sudah sesuai dengan karakteristiknya, di mana atribut seperti sepal_length, sepal_width, petal_length, dan petal_width bertipe numerik, sedangkan species bertipe kategorikal. <br>

Dari sisi outlier, meskipun terdapat beberapa nilai ekstrem secara alami, nilai tersebut masih berada dalam rentang yang wajar dan tidak menunjukkan kesalahan pencatatan. Format data juga konsisten dan tidak ditemukan inkonsistensi penulisan kategori. Dengan demikian, berdasarkan aspek kelengkapan, konsistensi, validitas, dan akurasi data, dataset ini masih dapat dikategorikan sebagai dataset yang baik dan layak digunakan untuk tahap analisis maupun pemodelan lebih lanjut.