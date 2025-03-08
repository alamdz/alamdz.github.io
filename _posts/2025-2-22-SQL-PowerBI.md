---
title: "Analisis Kinerja Penjualan Pizza Menggunakan SQL & Power BI"
description: Studi Kasus; Optimisasi Strategi Penjualan dengan Data-Driven Insights (Work in Progress).
date: 2025-3-8 00:00:00 +0000 #YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [Data Analyst, SQL, PowerBI] #[TOP_CATEGORIE, SUB_CATEGORIE]
tags: [data understanding, data cleaning, data visualization, filter, software] # TAG names should always be lowercase
image: /assets/img/sqlpowerbi.png #/path/to/image
alt: #"Image alt text"
---
<!-- https://www.youtube.com/watch?v=V-s8c6jMRN0 -->

## Pendahuluan
Analisis ini bertujuan memahami pola penjualan pizza menggunakan `SQL Server Management Studio` untuk ekstraksi data dan `Power BI` untuk visualisasi. Dataset mencakup informasi transaksi seperti tanggal, waktu, kategori, ukuran, dan pendapatan pizza.

**Tools & Metodologi**
- **SQL:** Ekstraksi dan transformasi data untuk menghitung KPI, tren, dan segmentasi penjualan.
- **Power BI:** Visualisasi interaktif untuk mengidentifikasi pola bisnis dan insight strategis.
- **Dataset:** `pizza_sales` (kolom: `order_id`, `order_date`, `order_time`, `pizza_name`, `pizza_category`, `pizza_size`, `quantity`, `total_price`)

>**Raw Dataset**: [**Pizza Sales (Kaggle)**](https://www.kaggle.com/datasets/nextmillionaire/pizza-sales-dataset){:target="_blank"}

## Analisis dengan SQL
### A. KPI’s
1. Total Revenue
```sql
SELECT FORMAT(SUM(total_price), 'C') AS Total_Revenue FROM pizza_sales;
```
- **Tujuan:** Mengetahui pendapatan total dari semua transaksi.
- **Output:** Nilai mata uang `$817,860.05`.

2. Average Order Value
Rata-rata nilai pesanan per transaksi (total pendapatan dibagi jumlah pesanan unik).
```sql
SELECT 
    (SELECT SUM(total_price) FROM pizza_sales) 
    / 
    (SELECT COUNT(DISTINCT order_id) FROM pizza_sales) 
    AS Avg_Order_Value;
```
- **Tujuan:** Menghitung rata-rata nilai belanja per transaksi.
- **Output:** Total Revenue / Jumlah Pesanan Unik = `38.307`.

3. Average Pizzas Per Order
Rata-rata jumlah pizza per pesanan (total pizza dibagi total pesanan).
```sql
WITH OrderSummary AS (
    SELECT 
        COUNT(DISTINCT order_id) AS total_orders,
        SUM(quantity) AS total_pizzas
    FROM pizza_sales
)
SELECT 
    CAST(total_pizzas AS DECIMAL(10,2)) / total_orders AS Avg_Pizzas_per_Order
FROM OrderSummary;
```
- **Tujuan:** Memahami kebiasaan pelanggan dalam memesan pizza.
- **Rumus:**Total Pizza Terjual / Jumlah Pesanan = `2.323`.

### B. Daily Trend for Total Orders
Menunjukkan distribusi jumlah pesanan per hari dalam seminggu (contoh: Senin, Selasa).
```sql
SELECT 
    DATENAME(DW, order_date) AS order_day,
    DATEPART(DW, order_date) AS day_number,
    COUNT(DISTINCT order_id) AS total_orders 
FROM pizza_sales
GROUP BY DATENAME(DW, order_date), DATEPART(DW, order_date)
ORDER BY day_number;
```
![Daily Trend](/assets/img/dailytrendsql.png){: width="220" height="200" .left}

- **Tujuan:** Mengidentifikasi hari dengan aktivitas pesanan tertinggi.
- **Insight:** Membantu optimasi stok bahan atau jadwal kerja karyawan.

### C. Monthly Trend for Orders
Menampilkan tren pesanan per bulan (contoh: Januari, Februari).
```sql
SELECT 
    DATENAME(MONTH, order_date) AS Month_Name,
    MONTH(order_date) AS Month_Number,
    COUNT(DISTINCT order_id) AS Total_Orders
FROM pizza_sales
GROUP BY DATENAME(MONTH, order_date), MONTH(order_date)
ORDER BY Month_Number;
```
![Monthly Trend](/assets/img/monthlytrendsql.png){: width="220" height="200" .center}
- **Tujuan:** Memantau fluktuasi penjualan per bulan.
- **Insight:** Deteksi musim tinggi (peak season) atau penurunan permintaan.

### D. % of Sales by Pizza Category
Persentase kontribusi pendapatan dari setiap kategori pizza (Classic, Veggie, Supreme, Chicken).
```sql
SELECT 
    pizza_category,
    ROUND(SUM(total_price) * 100.0 / SUM(SUM(total_price)) OVER(), 2) AS PCT
FROM pizza_sales
GROUP BY pizza_category;
```
![PercentPizza](/assets/img/percentpizza.png){: width="220" height="200" .center}

- **Tujuan:** Menilai kategori pizza paling menguntungkan.
- **Output:** `Classic: 26%` dari total pendapatan.

### E. % of Sales by Pizza Size
Persentase kontribusi pendapatan berdasarkan ukuran pizza (S, M, L, XL).
```sql
SELECT 
    pizza_size,
    ROUND(SUM(total_price) * 100.0 / (SELECT SUM(total_price) FROM pizza_sales), 2) AS PCT
FROM pizza_sales
GROUP BY pizza_size
ORDER BY 
    CASE pizza_size 
        WHEN 'S' THEN 1 
        WHEN 'M' THEN 2 
        WHEN 'L' THEN 3 
        WHEN 'XL' THEN 4 
    END;
```
![PercentPizza](/assets/img/pizzasize.png){: width="220" height="200" .center}

- **Tujuan:** Menentukan ukuran pizza paling populer.
- **Insight:** Menyesuaikan promosi berdasarkan preferensi pelanggan.

### F. Total Pizzas Sold by Pizza Category
Menampilkan jumlah pizza terjual per kategori
```sql
SELECT 
    pizza_category,
    SUM(quantity) AS Total_Pizzas_Sold,
    ROUND(SUM(quantity) * 100.0 / (SELECT SUM(quantity) FROM pizza_sales), 2) AS PCT
FROM pizza_sales
GROUP BY pizza_category;
```
![totalpizzacategory](/assets/img/totalpizzacategory.png){: width="250" height="200" .center}

- **Tujuan:** Mengetahui kategori pizza dengan volume penjualan tertinggi.

### G. Top/Bottom 5 Queries
5 Pizza dengan pendapatan tertinggi/terendah.
```sql
SELECT TOP 5 WITH TIES 
    pizza_name, 
    SUM(total_price) AS Total_Revenue
FROM pizza_sales
GROUP BY pizza_name
ORDER BY Total_Revenue DESC;
```
![topbottompizza](/assets/img/topbottompizza.png){: width="220" height="200" .center}

- **Tujuan:** Mengidentifikasi produk unggulan dan yang perlu perbaikan.

### H. Hourly Sales Trend
Analisis waktu pemesanan puncak:
```sql
SELECT 
    DATEPART(HOUR, order_time) AS hour_of_day,
    COUNT(DISTINCT order_id) AS total_orders
FROM pizza_sales
GROUP BY DATEPART(HOUR, order_time)
ORDER BY total_orders DESC;
```
![hourlysalespizza](/assets/img/hourlysalespizza.png){: width="220" height="200" .center}

- **Tujuan:** Mengetahui waktu puncak pemesanan.
- **Insight:** Optimasi jam operasional atau alokasi staf.

---

## Visualisasi dengan Power BI

**Tujuan:** Membangun dashboard interaktif untuk memantau kinerja penjualan pizza secara real-time.
1. **Integrasi Data**
- **Sumber Data:**
    - **Koneksi ke database SQL Server** untuk mengimpor tabel `pizza_sales`.
    - Proses: **`Home > Get Data > SQL Server`** di Power BI.
- **Data Cleaning:**
    - Menggunakan **Power Query Editor** untuk:
        - Mengganti nilai ukuran pizza:
        - `M → Medium`, `S → Regular`, `L → Large`, `XL → X-Large`.
        - Menghapus kolom tidak relevan atau duplikat.

2. **Pembuatan Measure & Kolom Terhitung (Measure Kunci (`DAX`)):**
- **Total Revenue:**
> `Total Revenue = SUM(pizza_sales[total_price])`
- **Total Orders:**
> `Total Orders = DISTINCTCOUNT(pizza_sales[order_id])`
- **Avg Order Value:**
> `Avg Order Value = DIVIDE([Total Revenue], [Total Orders])`
- **Total Pizzas Sold:**
> `Total Pizzas Sold = SUM(pizza_sales[quantity])`
- **Avg Pizza Per Order:**
> `Avg Pizza Per Order = DIVIDE([Total Pizzas Sold], [Total Orders])`

### **Desain Dashboard**
**Visualisasi Utama:**
- **KPI Card**
Menampilkan _Total Revenue , Total Orders , Avg Order Value , dan Avg Pizza Per Order._
- **Daily Trend**
    - **Column Chart:** Jumlah pesanan per hari (dengan filter hari).
- **Tren Bulanan**
    - **Area Chart:** Fluktuasi pesanan per bulan untuk analisis musiman.
- **% of Sales by Kategori Pizza**
    - Donut Chart: Persentase pendapatan per kategori (Classic, Veggie, dll.).
- **Total Pizza Sold by Category**
    - Stacked Bar Chart / Tunnel: Distribusi penjualan berdasarkan kategory.
- **Navigasi & Filter:**
    - **Slicer:** Pilih kategori pizza atau rentang tanggal.
    - **Page Navigator:** Beralih antar halaman dashboard (misal: "Harian" vs "Bulanan").

### Dashboard
![dashboard](/assets/img/powerbihome.png){: .shadow .rounded-40 .center}
![dashboard2](/assets/img/powerbihome2.png){: .shadow .rounded-40 .center}

### Insight dari Dashboard
1. *Polanya:*
- Penjualan tertinggi terjadi pada hari Jumat dan Kamis.
- Ukuran Large (L) mendominasi 45% total pendapatan.
2. **Rekomendasi:**
- Tambahkan promosi khusus untuk kategori Chicken Pizza yang kontribusinya masih rendah.
- Optimalkan stok bahan pada jam puncak (12:00–13:00 dan 18:00 - 19:00).

## Kesimpulan
Power BI memungkinkan transformasi data kompleks menjadi visualisasi intuitif yang mudah dipahami oleh stakeholder. Dengan dashboard ini, tim dapat:
- Mengambil keputusan cepat berdasarkan tren real-time.
- Mengidentifikasi peluang peningkatan penjualan.

**Langkah Selanjutnya:**
- Integrasi data real-time dengan API.
- Ekspor laporan otomatis ke email tim manajemen.