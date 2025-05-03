# Laporan Proyek Machine Learning - Carolus Christadi Cahyono

## Project Overview

Dalam era digital yang terus berkembang, kebutuhan akan sistem rekomendasi semakin penting, terutama dalam bidang hiburan seperti anime. Dengan semakin banyaknya pilihan anime yang tersedia, pengguna sering kali mengalami kesulitan dalam memilih tontonan yang sesuai dengan preferensi mereka. Oleh karena itu, diperlukan suatu sistem rekomendasi yang mampu membantu pengguna menemukan anime yang relevan berdasarkan kesamaan konten atau preferensi pengguna sebelumnya.

Proyek ini bertujuan untuk membangun sistem rekomendasi anime menggunakan pendekatan *Content-Based Filtering* dengan memanfaatkan data dari Kaggle: [Anime Recommendation Database](https://www.kaggle.com/datasets/CooperUnion/anime-recommendations-database). Pendekatan ini bekerja dengan menganalisis fitur dari anime itu sendiri (seperti genre) dan mencocokkannya dengan preferensi pengguna untuk memberikan rekomendasi yang relevan.

Menurut [Ricci et al., 2011], sistem rekomendasi berbasis konten sangat efektif digunakan ketika data historis interaksi pengguna terbatas atau preferensi pengguna baru belum banyak diketahui (*cold start*). Dengan pendekatan ini, sistem tetap dapat memberikan saran berdasarkan kemiripan fitur antar item.

## Business Understanding

### Problem Statements

- Bagaimana cara merekomendasikan anime kepada pengguna berdasarkan preferensi mereka terhadap anime yang pernah ditonton sebelumnya?
- Bagaimana cara mengukur kesamaan antar anime menggunakan fitur konten seperti genre?

### Goals

- Membangun model sistem rekomendasi anime berbasis *content-based filtering* menggunakan informasi konten (genre).
- Menghasilkan rekomendasi anime yang relevan bagi pengguna dengan mengukur kemiripan antar genre anime.

### Solution Statements

Untuk mencapai tujuan tersebut, pendekatan yang akan digunakan adalah sebagai berikut:

1. **Content-Based Filtering menggunakan Cosine Similarity**  
   Pendekatan ini akan menggunakan vektorisasi genre anime dalam bentuk *TF-IDF* atau *One-Hot Encoding*, kemudian menghitung kesamaan antar anime menggunakan *cosine similarity*. Anime yang paling mirip dengan anime yang disukai pengguna akan direkomendasikan.

---

**Referensi**:  
[1] Ricci, F., Rokach, L., & Shapira, B. (2011). *Introduction to Recommender Systems Handbook*. Springer.  
[2] Cooper Union. (n.d.). Anime Recommendations Database. Kaggle. Retrieved from https://www.kaggle.com/datasets/CooperUnion/anime-recommendations-database

## Data Understanding

Dataset yang digunakan dalam proyek ini adalah **Anime Recommendation Database** yang berasal dari Kaggle dan dapat diakses melalui tautan berikut: [Kaggle - Anime Recommendations Database](https://www.kaggle.com/datasets/CooperUnion/anime-recommendations-database).

Terdapat dua file utama:
1. **anime.csv**: Berisi informasi metadata terkait anime.
2. **rating.csv**: Berisi informasi rating yang diberikan oleh pengguna terhadap anime.

### Variabel pada anime.csv:
- `anime_id`: ID unik dari setiap anime di MyAnimeList.
- `name`: Nama lengkap dari anime.
- `genre`: Genre dari anime, dipisahkan dengan koma jika lebih dari satu.
- `type`: Tipe dari anime (TV, Movie, OVA, dll).
- `episodes`: Jumlah episode.
- `rating`: Rata-rata rating dari pengguna.
- `members`: Jumlah pengguna yang menambahkan anime ke daftar mereka.

### Variabel pada rating.csv:
- `user_id`: ID unik dari pengguna.
- `anime_id`: ID anime yang dirating oleh pengguna.
- `rating`: Nilai rating (1-10), atau -1 jika pengguna tidak memberikan rating eksplisit.

---

### Eksplorasi `anime.csv`

#### 1. `.head()`
Menampilkan 5 data teratas dari dataframe `anime_df`. Berguna untuk melihat struktur awal data.

#### 2. `.describe()`
Memberikan ringkasan statistik deskriptif terhadap kolom numerik seperti `rating` dan `members`.

**Insight:**
- Rata-rata rating anime adalah 6.47.
- Rating maksimal adalah 10, sedangkan minimum 1.67.
- Jumlah anggota komunitas (members) sangat bervariasi, dengan maksimum mencapai lebih dari 1 juta.

#### 3. `.info()`
Digunakan untuk melihat tipe data dan jumlah non-null.

**Insight:**
- Total ada 12.294 entri.
- Kolom `genre` memiliki 62 missing values.
- Kolom `type` memiliki 25 missing values.
- Kolom `rating` memiliki 230 missing values.
- Tipe data kolom `episodes` masih berupa string karena ada nilai seperti `'Unknown'`.

#### 4. `isnull().sum()`
Menampilkan jumlah missing values untuk setiap kolom.

**Insight tambahan:**
- Missing values pada `type` dan `rating` kemungkinan besar berasal dari anime yang belum tayang atau belum memiliki cukup data.

---

### Eksplorasi `rating.csv`

#### 1. `.head()`
Menampilkan 5 baris pertama dari `rate_df`.

**Insight:**
- Banyak pengguna memberikan rating `-1`, yang berarti mereka telah menonton namun tidak memberikan rating eksplisit.

#### 2. `.describe()`
Memberikan statistik deskriptif untuk kolom `user_id`, `anime_id`, dan `rating`.

**Insight:**
- Dataset terdiri dari 7.813.737 entri.
- Rating memiliki nilai minimum -1 dan maksimum 10.
- Rata-rata rating adalah 6.14 dengan standar deviasi 3.73, yang menunjukkan sebaran nilai rating cukup lebar akibat adanya nilai -1.

#### 3. `.info()`
Digunakan untuk melihat tipe data dan non-null count dari setiap kolom.

**Insight:**
- Semua kolom bertipe integer dan tidak memiliki missing values.

#### 4. `isnull().sum()`
Menampilkan 0 nilai kosong di seluruh kolom, berarti data lengkap secara struktur.

---

### Insight :

- Banyak nilai `rating = -1` pada `rating.csv` perlu di-filter sebelum proses pembobotan kemiripan dilakukan.
- Beberapa entri pada `anime.csv` seperti `episodes = "Unknown"` atau `type = NaN` perlu diproses atau diabaikan sesuai relevansi proyek.
- Distribusi genre cukup bervariasi dan dapat digunakan sebagai fitur penting untuk filtering.

### Visualisasi

Visualisasi data pada project ini bertujuan untuk memudahkan dan memahami data lebih lanjut. Oleh karena itu, dibuatlah visualisai sebagai berikut, 

#### Persebaran genre anime 

Data menggabungkan keseluruhan genre menjadi suatu string pada kolom sehingga sulit dibaca, oleh karena itu, dilakukan beberapa tahap sebagai berikut:

- Membuat genre sebagai list
- Melihat genre-genre yang ada dalam dataset
- Menghitung setiap genre
- Menampilkan dalam bentuk bar chart

#### Top 10 anime dengan rating tertinggi

Dataset yang diberikat pada rating.csv perlu di proses kembali agar bisa melihat total rating untuk satu anime sehingga dibuatlah langkah-langkah untuk dapat melakukan visualisasi

- Filter data dengan -1
- Melakukan grouping untuk setiap anime dengan anime_id yang sama dan melihat jumlah rating
- Mencari anime dengan 10 rating tertinggi
- Menampilkan dalam bentuk bar chart
