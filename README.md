# UTS Mata Kuliah Graf Sains Data

## Deskripsi Proyek
Proyek ini dikerjakan untuk memenuhi Ujian Tengah Semester (UTS) mata kuliah Graf Sains Data. Proyek berfokus pada penerapan konsep graf untuk menganalisis interaksi pengguna dengan produk pada dataset e-commerce RetailRocket menggunakan Python dan Neo4j. Data interaksi direpresentasikan sebagai graf yang terdiri dari node `Visitor` dan `Item`, serta relasi yang menggambarkan aktivitas pengguna terhadap produk. Analisis dilanjutkan dengan pembentukan Vector Space Model (VSM), proyeksi graf, perhitungan kemiripan menggunakan cosine similarity, dan sistem rekomendasi berbasis K-Nearest Neighbors (KNN).

## Dataset
Dataset yang digunakan adalah [RetailRocket E-commerce Dataset](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) dari Kaggle.
Dataset ini berisi aktivitas pengguna pada platform e-commerce, dengan file utama `events.csv` yang memiliki kolom berikut:
| Kolom | Keterangan |
|---|---|
| `timestamp` | Waktu terjadinya interaksi dalam format Unix timestamp milidetik |
| `visitorid` | ID pengguna atau Visitor |
| `event` | Jenis interaksi, seperti `view`, `addtocart`, dan `transaction` |
| `itemid` | ID produk atau Item |
| `transactionid` | ID transaksi, jika tersedia |

Daaset juga menyediakan file `category_tree.csv`, `item_properties_part1.csv`, dan `item_properties_part2.csv` yang berisi informasi kategori serta properti Item.

### Pembagian Data
Data interaksi dibagi secara kronologis menjadi dua periode:
- **P1 — Training:** 3 Mei 2015 sampai 5 Juni 2015.
- **P2 — Testing:** 6 Juni 2015 sampai 14 Juni 2015.

Batas waktu diterapkan dengan rentang awal inklusif dan akhir eksklusif.
| Dataset | Jumlah baris |
|---|---:|
| P1 — Training | 704.786 |
| P2 — Testing | 164.163 |
P1 digunakan untuk membangun graf, menghitung kemiripan, dan menghasilkan rekomendasi. P2 digunakan untuk menguji relevansi rekomendasi terhadap interaksi pengguna pada periode berikutnya.

## Tahapan Pengerjaan
### 1. Data Wrangling dan Exploratory Data Analysis
Data diperiksa, dibersihkan sesuai kebutuhan, dan difilter berdasarkan periode training dan testing. Analisis eksploratif dilakukan untuk memahami distribusi jenis interaksi, jumlah Visitor, jumlah Item, serta karakteristik data yang digunakan.

### 2. Pembentukan Graf di Neo4j
Data P1 dimasukkan ke Neo4j dengan struktur utama:
- Node `Visitor` untuk merepresentasikan pengguna.
- Node `Item` untuk merepresentasikan produk.
- Relasi `INTERACTED` untuk merepresentasikan interaksi pengguna dengan produk, dengan properti jenis event dan timestamp.
Hasil pemuatan data P1 terdiri dari 363.786 node Visitor, 125.585 node Item, dan 704.786 relasi interaksi.

### 3. Vector Space Model (VSM)
Matriks Visitor–Item dibentuk untuk merepresentasikan interaksi pengguna terhadap produk. Matriks tersebut kemudian digunakan untuk menghasilkan representasi vektor Visitor dan Item. Matriks disimpan dalam format sparse agar penggunaan memori lebih efisien karena sebagian besar pasangan Visitor–Item tidak memiliki interaksi.

### 4. Proyeksi Graf
Dua jenis proyeksi dianalisis:
- **Visitor Projection:** menghubungkan dua Visitor yang memiliki Item yang sama.
- **Item Projection:** menghubungkan dua Item yang berinteraksi dengan Visitor yang sama.
Karakteristik proyeksi dibandingkan berdasarkan jumlah node, jumlah edge, dan density. Visualisasi menggunakan NetworkX untuk membantu mengamati pola hubungan dalam graf.

### 5. Perhitungan Cosine Similarity
Cosine similarity digunakan untuk mengukur kemiripan vektor berdasarkan pola interaksi. Hasil kemiripan digunakan untuk menemukan Visitor yang memiliki pola interaksi serupa serta Item yang memiliki keterkaitan berdasarkan aktivitas pengguna. Hasil perhitungan similarity tertentu juga disimpan ke Neo4j melalui relasi `SIMILAR_TO`.

### 6. Sistem Rekomendasi KNN
Sistem rekomendasi dibangun dengan K-Nearest Neighbors menggunakan cosine distance dan nilai \(k=5\). Lima Visitor terdekat digunakan untuk membentuk kandidat rekomendasi berdasarkan Item yang pernah diakses tetangga, dengan mengecualikan Item yang sudah pernah diakses Visitor target pada P1.
Pada pengujian Visitor `222553`, KNN murni menghasilkan satu kandidat Item baru, yaitu `141572`. Karena jumlah kandidat terbatas, pendekatan tambahan berupa fallback popularitas Item dari P1 juga diuji. Pendekatan tambahan ini dibedakan dari hasil KNN murni.

### 7. Evaluasi Rekomendasi
Evaluasi dilakukan menggunakan data testing P2 dengan metrik berikut:
- **Hit@5:** mengukur apakah setidaknya satu Item relevan muncul dalam lima rekomendasi teratas.
- **NDCG@5:** mengukur relevansi sekaligus mempertimbangkan posisi Item relevan dalam daftar rekomendasi.
