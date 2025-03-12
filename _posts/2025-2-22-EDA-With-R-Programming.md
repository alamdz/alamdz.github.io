---
title: "Exploratory Data Analysis (EDA) Supply Chain Menggunakan R Programming"
description: Studi Kasus; menggunakan R Programming untuk Exploratory Data Analysis (EDA) pada data supply chain, bertujuan mengidentifikasi pola dan wawasan untuk meningkatkan efisiensi operasional.
date: 2025-2-22 00:00:00 +0000 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [Data Analyst, R Programming] #[TOP_CATEGORIE, SUB_CATEGORIE]
tags: [eda, data understanding, data cleaning, data visualization, r, supply chain, chi square] # TAG names should always be lowercase
image: /assets/img/edawithr.png #/path/to/image
alt: #"Image alt text"
---

## Pendahuluan
_Supply chain_ merupakan jaringan kompleks yang mencakup berbagai tahapan, mulai dari pengadaan bahan baku, produksi, hingga distribusi produk akhir ke konsumen. Dalam era digital saat ini, setiap tahap _Supply chain_ menghasilkan volume data yang besar, mulai dari data produksi, inventori, hingga logistik. Data ini memiliki potensi besar untuk diolah dan dianalisis guna meningkatkan efisiensi operasional, mengurangi biaya, dan memaksimalkan kepuasan pelanggan.

_Exploratory Data Analysis_ (EDA) menjadi langkah kritis dalam memahami pola, tren, dan hubungan antar variabel dalam data _supply chain_. Dengan menggunakan R Programming, yang dilengkapi berbagai paket seperti `tidyverse`, `ggplot2`, dan `dplyr`, kita dapat melakukan analisis data secara mendalam dan menyajikan visualisasi yang informatif. EDA tidak hanya membantu mengidentifikasi masalah dalam _Supply chain_, tetapi juga memberikan dasar untuk pengambilan keputusan yang lebih baik dan strategis.

## Langkah-langkah EDA dalam Supply Chain
### 1. Pengumpulan Data
Langkah pertama dalam EDA adalah mengumpulkan data yang relevan dari sumber terpercaya. Dataset yang digunakan dalam analisis ini diambil dari **Kaggle**, sebuah platform terbuka untuk berbagi dataset dan proyek data science. Dataset ini mencakup informasi rantai _Supply chain_ produk kecantikan (_Makeup Supply Chain Dataset_) dengan variabel seperti harga, tingkat defek, biaya produksi, dan moda transportasi.
>Dataset: [**Makeup Products Supply Chain Dataset**](https://www.kaggle.com/datasets/linusx/supply-chain-related-data/data){:target="_blank"}

Alasan Pemilihan Dataset:
- Memiliki variabel lengkap untuk analisis _Supply chain_ (produksi, distribusi, kualitas).
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
supply_chain <- janitor::clean_names(supply_chain)
```
- Memperbaiki penulisan kategori (misal: "`Product Type`" → "`product_type`") untuk memudahkan referensi kolom dalam `code`

3. Identifikasi Nilai Hilang
```R
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

4. Pengecekan Duplikasi Data
```R
jumlah_duplikat <- sum(duplicated(supply_chain))
cat("Jumlah baris duplikat:", jumlah_duplikat, "\n")
```
Output: `Jumlah baris duplikat: 0 `

5. Validasi Kategori
```R
unique(supply_chain$customer_demographics)
```
- Output: `"non-binary"` `"female"` `"unknown"` `"male"`
- Memastikan tidak ada kategori tidak valid (misal: "femal", "none").

6. Analisis Deskriptif Awal

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
### 4. Uji Chi-Square
- **Uji Chi-Square: Transportation Modes vs Inspection Results**
  - Tujuan: Menguji apakah ada hubungan signifikan antara moda transportasi (udara/laut/darat/rel) dengan hasil inspeksi (pass/fail).

```R
tabel_transport_inspect <- table(
  supply_chain$transportation_modes,
  supply_chain$inspection_results
)
hasil_chi <- chisq.test(tabel_transport_inspect)
print(hasil_chi)
```
Output: `X-squared = 0.52983, df = 6, p-value = 0.9975`

Interpetasi:
- p-value = 0.9975 (> 0.05): Tidak ada hubungan signifikan antara moda transportasi dan hasil inspeksi.
- Insight:
  - Hasil inspeksi (pass/fail) tidak dipengaruhi oleh moda transportasi.
  - Variasi hasil inspeksi lebih mungkin disebabkan oleh faktor lain, seperti kualitas produksi atau penanganan produk.

- **Uji Chi-Square: Product Type vs Location**
  - Tujuan: Menguji apakah ada hubungan signifikan antara kategori produk (skincare, haircare, cosmetics) dengan lokasi (Mumbai, Delhi, Kolkata, dll).

```R
tabel_transport_inspect <- table(
  supply_chain$transportation_modes,
  supply_chain$inspection_results
)
hasil_chi <- chisq.test(tabel_transport_inspect)
print(hasil_chi)
```
Output: `X-squared = 7.1186, df = 8, p-value = 0.5239`

Interpetasi:
- p-value = 0.5239 (> 0.05): Tidak ada hubungan signifikan antara kategori produk dan lokasi.
- Insight:
  - Distribusi produk tidak berbeda signifikan antar lokasi.
  - Kategori produk tersebar merata di semua lokasi, atau variasi yang ada hanya akibat kebetulan.

### 5. Kesimpulan dan Rekomendasi
**Kesimpulan**
1. Produk Skincare memiliki korelasi positif antara biaya produksi dan defect rate, serta defect rate tertinggi (2.45%).
2. Biaya Produksi Tinggi tidak menjamin revenue tinggi (korelasi negatif -0.21), bahkan cenderung meningkatkan defect rate.
3. Supplier 5 adalah penyumbang defect rate tertinggi, sementara Supplier 3 memberikan revenue terbaik.
4. Tidak Ada Hubungan Signifikan antara moda transportasi dan hasil inspeksi (p-value = 0.9975) atau lokasi dan kategori produk (p-value = 0.5239).

**Rekomendasi**
1. Optimasi Produksi Skincare:
  - Evaluasi proses produksi untuk mengurangi defect rate tanpa menaikkan biaya.
  - Prioritaskan kontrol kualitas ketat pada kategori ini.
2. Manajemen Biaya Produksi:
  - Fokus pada efisiensi produksi untuk menekan biaya tanpa mengorbankan kualitas.
3. Pemilihan Supplier Strategis:
  - Pertahankan kerja sama dengan Supplier 3 (revenue tinggi) dan evaluasi Supplier 5 (defect rate tinggi).
  - Gunakan Supplier 1 & 2 untuk penghematan biaya pengiriman.
4. Strategi Pengiriman:
  - Pilih moda transportasi berdasarkan biaya terendah (laut/rel), karena hasil inspeksi tidak terpengaruh.
5. Analisis Lanjutan:
  - Selidiki faktor non-transportasi yang memengaruhi defect rate (misal: penyimpanan, penanganan).