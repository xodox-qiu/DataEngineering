# DATA-ENGINEERING  
# Proyek : Analisis Keterkaitan Temperature (Suhu) Permukaan dengan Produksi Perikanan Air Tawar Tahunan

Proyek ini dikembangkan untuk menganalisis korelasi antara pola suhu permukaan dengan hasil produksi perikanan air tawar tahunan di Indonesia. Tujuan utama proyek adalah memahami hubungan antara suhu permukaan dan jumlah produksi ikan air tawar, serta memberikan rekomendasi untuk peningkatan efektivitas penggunaan atau penyesuaian suhu yang ada. Proyek ini mengintegrasikan data suhu permukaan dari Google Earth Engine (CFSR: Climate Forecast System Reanalysis) dan data produksi perikanan budidaya dari Badan Pusat Statistik (BPS) untuk tahun 2023, dengan analisis dilakukan per provinsi. Selain itu, proyek ini juga bertujuan menghasilkan laporan analisis yang mengidentifikasi korelasi antara suhu dan produksi ikan, serta mengidentifikasi pola perubahan tahunan untuk mendukung pengambilan keputusan oleh Kementerian Kelautan dan Perikanan.
---

## Manfaat Data / Use Case  
- **Tujuan Proyek:** Menyediakan data terintegrasi yang menggambarkan korelasi antara pola suhu permukaan dengan hasil produksi perikanan air tawar tahunan di Indonesia
- **Manfaat:**  
  -Menyediakan sumber data yang telah melalui proses validasi dan transformasi, berasal dari BPS dan Google Earth Engine, sehingga siap digunakan untuk analisis korelasi suhu dan produksi perikanan.
  - Membuka peluang bagi pengembang teknologi prediktif, seperti model machine learning untuk optimalisasi produksi perikanan berbasis suhu.
  - Hasil ETL proyek ini mendukung dashboard visualisasi untuk meningkatkan efisiensi analisis data perikanan dan memberikan rekomendasi penyesuaian suhu kepada Kementerian Kelautan dan Perikanan.

---

## Serving Analisis  
Data hasil ETL disimpan dalam format PostgreSQL yang bersih dan terstruktur. Data ini siap digunakan untuk analisis eksploratif serta visualisasi tren menggunakan perangkat lunak seperti Looker untuk menemukan pola dan korelasi.

## Serving Machine Learning  
Dataset bersih digunakan untuk membangun model regresi guna memprediksi produksi ikan nila tahun 2023 di Indonesia berdasarkan fitur Provinsi (diencode menggunakan LabelEncoder) dan Temperature Tahun 2023. Proyek ini mengimplementasikan Random Forest Regressor untuk prediksi produksi ikan nila. Model dievaluasi menggunakan metrik Mean Squared Error (MSE) dan R² Score, serta divalidasi melalui visualisasi scatter plot (prediksi vs aktual) dan plot feature importance untuk menganalisis kontribusi fitur terhadap prediksi.
---

## Transform ( Pembersihan & Transformasi )   
- **Pembersihan:**  
  - Hapus kolom kosong: Kolom seperti Gurame (2023) dan Ikan lainnya (2023) yang semuanya NaN dihapus menggunakan dropna(axis="columns", how='all').  
  - Hapus baris kurang data: Baris dengan kurang dari dua nilai non-NaN dihapus menggunakan dropna(axis='index', thresh=2).
  - Isi nilai hilang: Nilai NaN di kolom Nila (2023) diganti dengan "NO TANGKAP" menggunakan fillna.

- **Transformasi:**  
  - Gabungkan data: Dataset bps_df (produksi ikan) dan gee_df (suhu) digabungkan berdasarkan kolom Provinsi dengan merge(how='left').
  - Kelompokkan data: Data dikelompokkan berdasarkan Nila (2023), dengan menghitung rata-rata Suhu rata-rata dan menggabungkan daftar provinsi menggunakan groupby dan agg.
  - Ganti nama kolom: Kolom Nila (2023) diubah menjadi Ikan Nila 2023, dan Suhu 2023 menjadi Temperature Tahun 2023 menggunakan rename
  - Simpan data: 
  Data mentah diimpor dari produksi_ikan_bersih.csv dan suhu_perbandingan_celsius.csv.
  Data hasil transformasi disimpan ke new.csv (pemisah ;, desimal ,) dan dimuat ke database PostgreSQL menggunakan to_sql.
  

---

## Load ( Pemindahan ke Target ) 
- **Target:**  
  - Tabel baru bernama avnadmin di database PostgreSQL pada server Aiven, yang berfungsi sebagai output utama untuk analisis lebih lanjut oleh layanan lain.

- **Metode:**  
  - Simpan DataFrame df_hasil ke tabel avnadmin menggunakan to_sql()
    name="avnadmin": Nama tabel tujuan.
    con=engine: Koneksi SQLAlchemy ke database Aiven.
    if_exists="replace": Ganti tabel jika sudah ada.
    index=False: Abaikan indeks DataFrame.

  - Verifikasi dengan pd.read_sql() untuk mengambil 10 baris pertama dari tabel avnadmin menggunakan query SQL dengan LIMIT 10,

---

## Arsitektur / Workflow ETL  
- **Alur Modular:**  
  - Proses ETL diimplementasikan dalam pipeline modular yang terdiri dari fungsi-fungsi terpisah untuk setiap langkah, seperti membaca data, membersihkan, mengisi nilai hilang, menggabungkan, dan menyimpan data.
  - Pipeline utama dijalankan melalui fungsi pipeline() yang mengatur urutan eksekusi langkah-langkah ETL secara berurutan.
  - Proses ekstraksi data dari sumber eksternal diatur dalam fungsi seperti scrape_bps_data() dan extract_gee_data().
  - Transformasi data mencakup pembersihan, penggabungan data berdasarkan kolom Provinsi, dan pemformatan kolom seperti konversi suhu ke Celsius.
  - Data hasil transformasi disimpan ke file CSV (new.csv) dan dimuat ke database PostgreSQL menggunakan SQLAlchemy.

- **Tools yang Digunakan:**  
  - Python 3.x : Digunakan sebagai bahasa pemrograman utama.
  - Library : `pandas`, `selenium`, `ee`, `geemap`, `sqlalchemy`, `csv`, `sklearn`,'`seaborn`.
  - Environment : Notebook Jupyter
  - Database : PostgreSQL di Aiven Cloud untuk menyimpan data hasil ETL.
  - GeckoDriver untuk mendukung scraping dengan Selenium pada browser Firefox.

---

## Kode Program  
- **Struktur Kode:**  
  - Kode untuk ETL dan Machine Learning dipisahkan ke dalam dua notebook terpisah untuk menjaga modularitas dan kejelasan.
  - Penamaan variabel dan fungsi bersifat deskriptif (contoh: bps_df, gee_df, final_df, scrape_bps_data, transform_data) untuk memudahkan pemahaman dan pemeliharaan kode.
    
- **Machine Learning:**  
  - Menggunakan model Random Forest Regressor untuk memprediksi produksi ikan nila tahun 2023 berdasarkan fitur Provinsi (diencode menggunakan LabelEncoder) dan Temperature Tahun 2023.
  - Model dievaluasi menggunakan metrik Mean Squared Error (MSE) dan R² Score, serta divisualisasikan melalui scatter plot prediksi vs aktual dan plot feature importance untuk mengidentifikasi kontribusi fitur terhadap prediksi.  

---

## Kontributor

| Nama Lengkap                     | NIM         | Peran                |
|----------------------------------|-------------|----------------------|
| Alif Rahmathul J                 | 234311030   | Data Analyst         |
| Fajar Hakiki                     | 234311041   | Data Engineer        |
| Frezy Ananta D. Tertiya          | 234311045   | Project Manager      |
| Lyan Fairus                  	   | 234311050   | Data Analyst    	|
