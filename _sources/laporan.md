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

Proyek ini menggunakan data time series dari data historis perdagangan saham harian perusahaan Adaro (ADARO) selama 5 tahun dari 22 Agustus 2019 sampai 20 September 2024 yang didapat dari situs web [Google Finance](https://www.google.com/finance/quote/ADRO:IDX?hl=en&window=5Y) yang datanya bersumber langsung dari Bursa Efek Indonesia yang bertanggung jawab dalam menyediakan semua sarana perdagangan efek dan membuat peraturan yang berkaitan dengan kegiatan bursa di Indonesia. Data yang diambil berformat CSV (Comma-separated value) yang berisikan 1234 baris perdagangan saham.

```{code-cell}
#import library
import pandas as pd
```
```{code-cell}
# Load data
df = pd.read_csv('https://raw.githubusercontent.com/atsugaa/psd/refs/heads/main/ADRO.csv', parse_dates = True, low_memory = False, index_col = 'Date')
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


```{code-cell}
df['Volume']
```
```{code-cell}
df.dtypes
```
```{code-cell}
df.describe()
```
```{code-cell}
df.info()
```

Terbaca hanya 5 kolom karena kolom Date (tanggal) telah dijadikan indeks untuk memudahkan nantinya.

#### Eksplorasi Data

Melihat tiap tiap kolom yang memiliki nilai 0

```{code-cell}
#Data Open yang memiliki nilai 0
zero_open = df[df.Open == 0]
print("In total: ", zero_open.shape)
df.Open.isna().sum()
zero_open.head(5)
```

Terlihat bahwa kolom Open tidak memiliki nilai 0


```{code-cell}
#Data High yang memiliki nilai 0
zero_high = df[df.High == 0]
print("In total: ", zero_high.shape)
df.High.isna().sum()
zero_high.head(5)
```

Terlihat bahwa kolom High tidak memiliki nilai 0


```{code-cell}
#Data Low yang memiliki nilai 0
zero_low = df[df.Low == 0]
print("In total: ", zero_low.shape)
df.Low.isna().sum()
zero_low.head(5)
```

Terlihat bahwa kolom Low tidak memiliki nilai 0

```{code-cell}
#Data Close yang memiliki nilai 0
zero_close = df[df.Close == 0]
print("In total: ", zero_close.shape)
df.Close.isna().sum()
zero_close.head(5)
```

Terlihat bahwa kolom Close tidak memiliki nilai 0

```{code-cell}
#Data Volume yang memiliki nilai 0
zero_volume = df[df.Volume == 0]
print("In total: ", zero_volume.shape)
zero_volume
```

Terlihat bahwa kolom Volume tidak memiliki nilai 0.

Selanjutnya melihat apakah data terdapat outlier

```{code-cell}
import numpy as np
import matplotlib.pyplot as plt
numeric_cols = df
fig, ax = plt.subplots()
ax.boxplot(numeric_cols)
df.describe()
```

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


#### Verifikasi Kualitas Data

Pada tahap ini dilakukan evaluasi terhadap data untuk memastikan bahwa data cocok untuk digunakan, apa saja kekurangan data, tindakan apa yang diperlukan untuk mengatasi kekurangan dari data

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
