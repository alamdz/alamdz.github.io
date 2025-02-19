---
title: "Exploratory Data Analysis (EDA) Supply Chain Menggunakan R Programming (On Progress)"
description: Proyek ini menggunakan R Programming untuk Exploratory Data Analysis (EDA) pada data rantai pasokan, bertujuan mengidentifikasi pola dan wawasan untuk meningkatkan efisiensi operasional.
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
Proses pembersihan data dilakukan untuk memastikan kualitas dan konsistensi dataset sebelum analisis. Berikut langkah-langkah yang telah diimplementasikan: dan konsistensi analisis. Tahap ini mencakup:

1. Membaca Data
```R
# Membaca file CSV dari lokasi spesifik
supply_chain <- read_csv("C:/Alam/Pratice/Project/supply_chain_dataset.csv")
```
- Tujuan: Memuat dataset dari sumber lokal.

2. Membersihkan Nama Kolom
```R
# Standarisasi format nama kolom (snake_case)
supply_chain <- janitor::clean_names(supply_chain)
```
- Memperbaiki penulisan kategori (misal: "`Product Type`" → "`product_type`") untuk memudahkan referensi kolom dalam `code`

3. Identifikasi Nilai Hilang
```R
# Cek missing values per kolom
colSums(is.na(supply_chain))
```
Output:
```r
product_type               0  
sku                        0  
price                      0  
...  
defect_rates               0  
```
- Insight: Tidak ada nilai hilang (_missing values_) pada dataset ini.

4. Validasi Kategori
```R
# Cek kategori unik
unique(supply_chain$customer_demographics)
```
- Output: `"non-binary"` `"female"` `"unknown"` `"male"`
- Memastikan tidak ada kategori tidak valid (misal: "femal", "none").

5. Analisis Deskriptif Awal

```R
product_distribution <- supply_chain %>%
  group_by(product_type) %>%
  summarise(
    total_sku = n(),
    avg_defect_rate = mean(defect_rates, na.rm = TRUE),
    avg_revenue = mean(revenue_generated, na.rm = TRUE)
  )
```
Output:
```R
  product_type total_sku avg_defect_rate avg_revenue
  <chr>            <int>           <dbl>       <dbl>
1 cosmetics           26            1.92       6212.
2 haircare            34            2.48       5131.
3 skincare            40            2.33       6041.
```
Insight:
- `haircare` memiliki revenue terendah tetapi defect rate tertinggi.
- `cosmetics` memiliki defect rate terendah dengan revenue tertinggi.

### 3. Visualisasi Data
Visualisasi data adalah langkah kritis dalam EDA untuk memahami pola, tren, dan hubungan antar variabel dalam dataset. Berikut adalah beberapa visualisasi yang telah dibuat untuk menganalisis dataset supply chain produk _makeup_:

- **Scatter Plot dengan Regresi & Facet Wrap**
![Scatter Plot](/assets/img/scatterplot.png){: width="972" height="589" .w-50 .right}
  - Tujuan: Menganalisis hubungan antara defect rate dan biaya produksi untuk setiap kategori produk.
  - Insight: 
    - Cosmetics: Korelasi negatif, semakin tinggi defect rate, semakin rendah biaya produksi. biaya produksi rendah cenderung memiliki defect rate lebih tinggi.
    - Haircare: Tidak ada korelasi jelas, kemungkinan dipengaruhi faktor lain.
    - Skincare: Korelasi positif, semakin tinggi biaya produksi, semakin tinggi defect rate. biaya produksi tinggi cenderung memiliki defect rate lebih tinggi.

```R
ggplot(supply_chain, aes(x = defect_rates, y = manufacturing_costs, color = product_type)) +
  geom_point(alpha = 0.7, size = 3) +
  geom_smooth(method = "lm", se = FALSE) +
  facet_wrap(~product_type, scales = "free") +
  labs(
    title = "Defect Rate vs. Biaya Produksi per Kategori Produk",
    x = "Defect Rate (%)",
    y = "Biaya Produksi (USD)",
    color = "Tipe Produk"
  ) +
  theme_minimal() +
  scale_color_brewer(palette = "Set1")
```
- **Heatmap Korelasi**
![Heatmap](/assets/img/heatmapkorelasi.png){: width="972" height="589" .w-50 .right}
  - Tujuan: Mengidentifikasi hubungan antar variabel numerik seperti harga, biaya produksi, dan revenue.
  - Insight: 
    - Manufacturing Costs vs Revenue Generated (-0.21): Hubungan negatif lemah, biaya produksi tinggi tidak selalu menghasilkan pendapatan lebih besar.
    - Defect Rates vs Revenue Generated (-0.13): Defect rate tinggi sedikit mengurangi pendapatan.

```R
numeric_vars <- supply_chain %>%
  select(price, manufacturing_costs, defect_rates, revenue_generated, shipping_costs)

cor_matrix <- cor(numeric_vars, use = "complete.obs")

corrplot(cor_matrix, 
         method = "color", 
         type = "upper", 
         tl.col = "black",
         addCoef.col = "black",
         number.cex = 0.7,
         title = "Korelasi Variabel Numerik")
```
- **Violin Plot + Strip Chart**
![Violin Plot](/assets/img/violinplot.png){: width="972" height="589" .w-50 .right}
  - Tujuan: Menganalisis distribusi biaya pengiriman per moda transportasi.
  - Insight:
    - Air: Rentang biaya pengiriman relatif tinggi dengan variasi yang luas.
    - Rail: Biaya pengiriman cenderung moderat, dengan sebaran yang lebih sempit.
    - Road: Bervariasi di kisaran menengah, terlihat tumpang tindih dengan Rail.
    - Sea: Umumnya paling rendah, namun tetap ada beberapa nilai yang lebih tinggi.

```R
ggplot(supply_chain, aes(x = transportation_modes, y = shipping_costs, fill = transportation_modes)) +
  geom_violin(alpha = 0.6) +
  geom_jitter(width = 0.1, alpha = 0.5, color = "black") +
  labs(
    title = "Distribusi Biaya Pengiriman per Moda Transportasi",
    x = "Moda Transportasi",
    y = "Biaya Pengiriman (USD)"
  ) +
  theme_minimal() +
  scale_fill_brewer(palette = "Set2")
```
- **Bubble Chart Multidimensi**
![Bubble Chart](/assets/img/bubblechart.png){: width="972" height="589" .w-50 .right}
  - Tujuan: Menyelidiki hubungan antara defect rate, revenue, volume produksi, dan lokasi.
  - Insight:
    - Defect rate berkisar antara 1.5% - 3%, dengan variasi revenue yang cukup luas.
    - Kota dengan produksi lebih besar (gelembung lebih besar) cenderung memiliki revenue lebih tinggi.
    - Delhi & Mumbai (hijau & kuning) mendominasi produksi dengan revenue tinggi meskipun defect rate bervariasi.
    - Beberapa titik dengan defect rate lebih tinggi (>2.5%) memiliki revenue lebih rendah, menunjukkan potensi dampak defect terhadap pendapatan.

```R
supply_chain %>%
  group_by(location, product_type) %>%
  summarise(
    avg_revenue = mean(revenue_generated),
    avg_defect = mean(defect_rates),
    total_production = sum(production_volumes)
  ) %>%
  ggplot(aes(x = avg_defect, y = avg_revenue, size = total_production, color = location)) +
  geom_point(alpha = 0.8) +
  labs(
    title = "Analisis 4D: Defect Rate, Revenue, Produksi, & Lokasi",
    x = "Rata-Rata Defect Rate (%)",
    y = "Rata-Rata Revenue (USD)",
    size = "Total Produksi",
    color = "Lokasi"
  ) +
  theme_minimal() +
  scale_color_viridis_d()
```
- **Radar Chart untuk Analisis Supplier**
![Radar Chart](/assets/img/radarchart.png){: width="972" height="589" .w-50 .right}
  - Tujuan: Membandingkan performa supplier berdasarkan defect rate, lead time, biaya pengiriman, dan revenue.
  - Insight: 
    - Supplier 5 memiliki defect rate tertinggi, yang bisa berdampak pada kualitas.
    - Supplier 4 memiliki lead time paling lama, yang bisa mempengaruhi efisiensi supply chain.
    - Supplier 3 menunjukkan revenue tertinggi, berpotensi sebagai supplier dengan performa terbaik.
    - Supplier 1 & 2 memiliki biaya pengiriman yang relatif rendah, dapat menjadi opsi hemat biaya.

```R
supplier_stats <- supply_chain %>%
  group_by(supplier_name) %>%
  summarise(
    avg_defect = mean(defect_rates),
    avg_lead_time = mean(lead_times),
    avg_shipping_cost = mean(shipping_costs),
    avg_revenue = mean(revenue_generated)
  ) %>%
  mutate(across(-supplier_name, scales::rescale))  # Normalisasi data

ggradar(supplier_stats, axis.label.size = 4, grid.label.size = 4) +
  labs(title = "Perbandingan Performa Supplier")
```
### 4. Analisis Deskriptif
a <!-- Analisis deskriptif memberikan ringkasan statistik dari data, seperti mean, median, modus, dan standar deviasi. Ini membantu dalam memahami karakteristik dasar dari data yang ada. --> 

### 5. Identifikasi Pola dan Tren
a <!-- Dengan menggunakan analisis waktu dan teknik statistik lainnya, kita dapat mengidentifikasi pola dan tren dalam data _supply chain_. Ini dapat mencakup analisis musiman, tren jangka panjang, dan fluktuasi permintaan. --> 

### 6. Kesimpulan dan Rekomendasi
a <!-- Setelah melakukan EDA, penting untuk menyusun kesimpulan dan rekomendasi berdasarkan temuan. Ini dapat mencakup saran untuk perbaikan proses, pengurangan biaya, atau peningkatan layanan pelanggan. --> 