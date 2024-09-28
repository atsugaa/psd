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


# Laporan Proyek Sains Data

## Pendahuluan

### Latar Belakang

Adaro minerals Indonesia merupakan perusahaan pertambangan indonesia yang memiliki visi menjadi perusahaan pertambangan indonesia yang terkemuka. Perusahaan ini memiliki salah satu misi yaitu memaksimalkan nilai bagi pemegang saham. Untuk dapat mencapai visi misi perusahaan diperlukan untuk mengetahui harga saham perusahaan kedepannya.

Saham adalah surat berharga yang menunjukkan kepemilikan seseorang dalam suatu perusahaan. Tujuan perusahaan menerbitkan saham adalah untuk mendapatkan dana dari pihak yang ingin menanamkan modal untuk mendapatkan keuntungan di masa depan, dana tersebut dapat digunakan untuk mengembangkan bisnis di perusahaan. Saham bersifat fluktuatif, bisa naik bisa turun sama halnya dengan harga barang atau komoditi di pasar. Bagi beberapa orang disanalah seninya, bila pasar statis tidak akan menarik minat investor. Naik turunnya harga saham juga dapat dipengaruhi oleh kinerja perusahaan, semakin baik kinerja perusahaan, maka dapat dipastikan harga saham perusahaan juga naik.

### Masalah

Adaro minerals indonesia yang memiliki salah satu misi yaitu memaksimalkan nilai bagi pemegang saham. Untuk dapat mencapai misi perusahaan tersebut diperlukan untuk mengetahui harga saham perusahaan kedepannya. Selain itu juga, harga saham yang menurun dalam jangka waktu yang lama dapat membuat investor menjadi hilang kepercayaan kepada perusahaan, hal itu dapat menghambat perkembangan perusahaan. Oleh karena itu muncul ide untuk melakukan peramalan harga saham dimasa mendatang.

### Tujuan

Dengan dilakukannya peramalan harga saham x minerals indonesia, diharapkan dapat menjadi indikator kinerja perusahaan dan perusahaan dapat mengantisipasi jika diramalkan harga saham turun dikemudian hari, dengan begitu kinerja perusahaan dapat selalu baik dan harga saham dapat terjaga. Hal ini dapat mencapai salah satu misi perusahaan yaitu memaksimalkan nilai bagi pemegang saham. Dan akan ada banyak investor yang tertarik untuk berinvestasi pada perusahaan, hal ini dapat membantu pendanaan perusahaan untuk dapat lebih berkembang.


## Metodologi

### Memahami Data (Data Understanding)

#### Pengumpulan Data

Proyek ini menggunakan data time series dari data historis perdagangan saham harian perusahaan Adaro (ADARO) selama hari kerja dalam jangka waktu 5 tahun dari 22 Agustus 2019 sampai 20 September 2024 yang didapat dari situs web [Google Finance](https://www.google.com/finance/quote/ADRO:IDX?hl=en&window=5Y) yang datanya bersumber langsung dari Bursa Efek Indonesia yang bertanggung jawab dalam menyediakan semua sarana perdagangan efek dan membuat peraturan yang berkaitan dengan kegiatan bursa di Indonesia. Data yang diambil berformat CSV (Comma-separated value) yang berisikan 1234 baris perdagangan saham.

```{code-cell}
#import library
import pandas as pd
```
```{code-cell}
# Load data
df = pd.read_csv('https://raw.githubusercontent.com/atsugaa/psd/refs/heads/main/ADRO.csv')
pd.options.display.float_format = '{:.0f}'.format
df.head()
```

#### Deskripsi Data

Data saham berupa csv berisikan 1234 baris dan 6 kolom yang merupakan :

- Date : Tanggal perdagangan saham yang berformat yyyy-mm-dd hh:mm:ss
- Open : Harga saat pasar saham dibuka ditanggal tertentu (pukul 9 pagi)
- High : Harga tertinggi yang pernah dicapai ditanggal tertentu
- Low : Harga terendah yang pernah dicapai ditanggal tertentu
- Close : Harga terakhir saham dalam rupiah saat pasar saham ditutup hari itu (pukul 3 sore)
- Volume : Besaran transaksi yang terjadi ditanggal tersebut dalam jutaan


Melihat tipe data dari masing masing kolom.

```{code-cell}
df.dtypes
```
```{code-cell}
df.describe()
```
```{code-cell}
df.info()
```
Terbaca bahwa dari 1234 baris data tiap kolomnya tidak ada baris yang kosong (null).


#### Eksplorasi Data

Sebelum melanjutkan eksplorasi data, terlebih dahulu menghapus jam, menit, dan detik dari kolom tanggal (Date) dan menjadikannya sebagai index dari data agar memudahkan prosesnya, Hasilnya seperti dibawah.

```{code-cell}
df['Date'] = pd.to_datetime(df['Date'], dayfirst=True).dt.date
df.set_index('Date', inplace=True)
df.index = pd.to_datetime(df.index)
df.head()
```


Selanjutnya melihat apakah data terdapat outlier

```{code-cell}
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sb
features = ['Open', 'High', 'Low', 'Close', 'Volume']
plt.subplots(figsize=(20,10))
for i, col in enumerate(features):
  plt.subplot(2,3,i+1)
  sb.boxplot(df[col])
plt.show()
```

Terlihat hanya kolom Volume yang memiliki outlier.
Selanjutnya melihat data trend di masing-masing kolom


```{code-cell}
#Data Trend
for i in df:
  df[i].plot(kind='line', figsize=(8, 4), title=i)
  plt.show()
```

Selanjutnya melihat korelasi antara kolom satu dengan kolom lainnya


```{code-cell}
import seaborn as sns
df_corr = df.corr()

# Menyiapkan gambar matplotlib
f, ax = plt.subplots(figsize=(11, 9))

# Membuat heatmap dengan mask dan proporsi aspek yang benar
sns.heatmap(df_corr, square=True, annot=True, linewidths=0.5, ax=ax, cmap="BuPu")
plt.show()
```

Terlihat tiap kolom kecuali kolom Volume memiliki nilai korelasi yang sangat tinggi (0.99-1), dapat diartikan bahwa tiap tiap kolom saling berhubungan satu sama lain kecuali kolom volume yang memiliki nilai korelasi negatif dengan kolom lainnya. Fitur yang ingin diprediksi adalah Close karena dengan mengetahui harga close di esok hari maka bisa menjadi sinyal untuk membeli saham jika harga prediksi close lebih besar dari harga close hari ini dan sebaliknya, maka perlu dilakukan seleksi fitur.

```{code-cell}
train_df = df.filter(['Close'])
train_df
```

### Pra-pemrosesan Data (Data Preprocessing)

#### Memilih Data

Sebelum data digunakan, data akan dipilih terlebih dahulu bagian mana saja yang akan digunakan nantinya serta alasan alasan mengapa bagian tersebut digunakan atau mengapa bagian tersebut tidak digunakan.

#### Membersihkan Data

Pada tahap ini dilakukan pembersihan data sebelum digunakan seperti melakukan penghapusan data maupun baris / kolom yang tidak relevan atau tidak akurat. Pada data cleaning juga menghapus data duplikat untuk menghindari pengaruh analisis yang tidak sesuai.

### Pemodelan Data (Data Modelling)

#### Memilih Model

Memilih model yang paling cocok sesuai dengan topik yang dibawa.

#### Membangun Model

Membangun model yang sudah ditentukan tekniknya sebelumnya

#### Menilai Model

Mengevaluasi model yang telah dibuat, menjelaskan kekurangan model, dan hal hal yang perlu diperbaiki untuk model nantinya

### Evaluasi

Evaluasi akhir yang menjelaskan nilai akhir dari proyek, menjelaskan hasil, kesalahan yang mungkin telah dilakukan saat proses, serta kekurangan dari proyek.

### Deployment

Bagian ini merupakan langkah terakhir di mana model yang sudah dievaluasi dan dianggap cukup baik diimplementasikan seperti aplikasi web, aplikasi mobile dan lain-lain. Deployment dilakukan setelah hasil model dianalisis dan dianggap layak untuk digunakan.

## Penutup

Berisikan ucapan penutup dari laporan proyek sains data.

## Daftar Rujukan

Berisikan rujukan rujukan yang membantu saat melakukan proyek sains data.
