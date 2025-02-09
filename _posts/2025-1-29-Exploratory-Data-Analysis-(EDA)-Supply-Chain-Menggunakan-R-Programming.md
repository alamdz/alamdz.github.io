---
title: "Exploratory Data Analysis (EDA) Supply Chain Menggunakan R Programming (On Progress)"
date: 2025-1-29 00:00:00 +0000 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [Data Analyst, R Programming] #[TOP_CATEGORIE, SUB_CATEGORIE]
tags: [eda, data, r, supply chain] # TAG names should always be lowercase
image: /assets/img/reda.png #/path/to/image
alt: #"Image alt text"
---

## Pendahuluan
_Supply chain_ adalah jaringan yang kompleks yang melibatkan berbagai proses, mulai dari pengadaan bahan baku hingga distribusi produk akhir kepada konsumen. Dengan meningkatnya volume data yang dihasilkan dalam setiap tahap rantai pasokan, penting untuk melakukan analisis yang mendalam untuk mengoptimalkan proses dan pengambilan keputusan. `R Programming`, dengan berbagai paket dan fungsionalitasnya, menyediakan alat yang kuat untuk melakukan EDA.

## Langkah-langkah EDA dalam Supply Chain
### 1. Pengumpulan Data
Langkah pertama dalam EDA adalah mengumpulkan data yang relevan dari sumber terpercaya. Dataset yang digunakan dalam analisis ini diambil dari **Kaggle**, sebuah platform terbuka untuk berbagi dataset dan proyek data science. Dataset ini mencakup informasi rantai pasok produk kecantikan (_Makeup Supply Chain Dataset_) dengan variabel seperti harga, tingkat defek, biaya produksi, dan moda transportasi.
>Dataset: [**Makeup Products Supply Chain Dataset**](https://www.kaggle.com/datasets/linusx/supply-chain-related-data/data)

Alasan Pemilihan Dataset:
- Memiliki variabel lengkap untuk analisis rantai pasok (produksi, distribusi, kualitas).
- Data terkini dan relevan dengan industri kecantikan yang sedang berkembang.
- Tersedia secara terbuka dan telah divalidasi oleh komunitas Kaggle.

### 2. Pembersihan Data
Setelah data dikumpulkan, langkah kritis berikutnya adalah pembersihan data (_data cleaning_) untuk memastikan kualitas dan konsistensi analisis. Tahap ini mencakup:

1. Penanganan Nilai Hilang (_Missing Values_):
  - Menggunakan fungsi `colSums(is.na(data))` di R untuk mengidentifikasi kolom dengan nilai kosong.
  - Strategi: Imputasi nilai rata-rata atau penghapusan baris jika nilai hilang <5%.

2. Standarisasi Format:
  - Mengubah semua teks menjadi huruf kecil (_lowercase_) untuk konsistensi kategori:

```R
data <- data %>%  
  mutate(  
    product_type = tolower(product_type),  
    customer_demographics = tolower(customer_demographics)  
  )  
```

> Memperbaiki kesalahan penulisan kategori (misal: "`Non-Binary`" → "`non-binary`").
{: .prompt-info }

- Penghapusan Duplikat:

```r
data <- distinct(data)  # Menghapus baris duplikat  
```
- Pengecekan Outlier:
  - Visualisasi boxplot untuk variabel numerik seperti `manufacturing_costs` atau `defect_rates`
  - Penanganan: Transformasi logaritmik atau _Winsorizing_ untuk outlier ekstrem.

Contoh Output:
```r
# Cek missing values setelah cleaning  
colSums(is.na(data))  

# Hasil:  
# product_type              0  
# defect_rates              0  
# manufacturing_costs       0  
```

### 3. Visualisasi Data
Visualisasi adalah bagian penting dari EDA. Dengan menggunakan paket seperti ggplot2, kita dapat membuat berbagai jenis grafik untuk memahami distribusi data, hubungan antar variabel, dan tren waktu. Contoh visualisasi yang dapat dilakukan meliputi:
- Histogram untuk melihat distribusi variabel
- Boxplot untuk mendeteksi outlier
- Scatter plot untuk menganalisis hubungan antar variabel

### 4. Analisis Deskriptif
a <!-- Analisis deskriptif memberikan ringkasan statistik dari data, seperti mean, median, modus, dan standar deviasi. Ini membantu dalam memahami karakteristik dasar dari data yang ada. --> 

### 5. Identifikasi Pola dan Tren
a <!-- Dengan menggunakan analisis waktu dan teknik statistik lainnya, kita dapat mengidentifikasi pola dan tren dalam data _supply chain_. Ini dapat mencakup analisis musiman, tren jangka panjang, dan fluktuasi permintaan. --> 

### 6. Kesimpulan dan Rekomendasi
a <!-- Setelah melakukan EDA, penting untuk menyusun kesimpulan dan rekomendasi berdasarkan temuan. Ini dapat mencakup saran untuk perbaikan proses, pengurangan biaya, atau peningkatan layanan pelanggan. --> 