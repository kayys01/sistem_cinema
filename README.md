# 🎬 DB_CINEMA

### Implementasi Database Relasional menggunakan MariaDB

Project ini merupakan contoh implementasi **database relasional sederhana** yang bertema sistem informasi perfilman. Database dibuat menggunakan **MariaDB (melalui XAMPP)** dan bertujuan untuk memahami konsep dasar pengelolaan data dalam sistem basis data.

Database ini menunjukkan beberapa konsep utama seperti:

* Pembuatan tabel relasional
* Penggunaan **Primary Key** dan **Foreign Key**
* Operasi penggabungan tabel menggunakan **JOIN**
* Pengelolaan data film, sutradara, dan genre

---

# 🏗️ 1️⃣ Struktur Database

## 📌 Membuat Database

```sql
CREATE DATABASE db_cinema;
USE db_cinema;
```

---

## 🎥 Tabel `sutradara`

Tabel ini menyimpan data sutradara film.

```sql
CREATE TABLE sutradara (
    id_sutradara INT PRIMARY KEY AUTO_INCREMENT,
    nama_sutradara VARCHAR(100),
    negara_asal VARCHAR(50),
    tahun_lahir INT
);
```

### Contoh Data Sutradara

| id_sutradara | nama_sutradara    | negara_asal     | tahun_lahir |
| ------------ | ----------------- | --------------- | ----------- |
| 1            | Christopher Nolan | Inggris         | 1970        |
| 2            | Joko Anwar        | Indonesia       | 1976        |
| 3            | Bong Joon-ho      | Korea Selatan   | 1969        |
| 4            | Steven Spielberg  | Amerika Serikat | 1946        |
| 5            | Greta Gerwig      | Amerika Serikat | 1983        |

---

## 🎭 Tabel `genre`

Tabel ini menyimpan kategori atau jenis film.

```sql
CREATE TABLE genre (
    id_genre INT PRIMARY KEY AUTO_INCREMENT,
    nama_genre VARCHAR(50),
    keterangan TEXT
);
```

### Contoh Data Genre

| id_genre | nama_genre | keterangan                     |
| -------- | ---------- | ------------------------------ |
| 1        | Action     | Film dengan banyak adegan aksi |
| 2        | Drama      | Cerita konflik kehidupan       |
| 3        | Thriller   | Film penuh ketegangan          |
| 4        | Sci-Fi     | Fiksi ilmiah dan teknologi     |
| 5        | Horror     | Film dengan unsur menakutkan   |

---

## 🎞️ Tabel `film`

Tabel ini berisi data film yang ditayangkan serta relasinya dengan tabel **sutradara** dan **genre**.

```sql
CREATE TABLE film (
    id_film INT PRIMARY KEY AUTO_INCREMENT,
    judul_film VARCHAR(150),
    tahun_rilis INT,
    durasi_menit INT,
    id_sutradara INT,
    id_genre INT,
    FOREIGN KEY (id_sutradara) REFERENCES sutradara(id_sutradara),
    FOREIGN KEY (id_genre) REFERENCES genre(id_genre)
);
```

### Contoh Data Film

| id_film | judul_film     | tahun_rilis | durasi_menit | id_sutradara | id_genre |
| ------- | -------------- | ----------- | ------------ | ------------ | -------- |
| 1       | Inception      | 2010        | 148          | 1            | 4        |
| 2       | Pengabdi Setan | 2017        | 107          | 2            | 5        |
| 3       | Parasite       | 2019        | 132          | 3            | 2        |
| 4       | Jurassic Park  | 1993        | 127          | 4            | 1        |
| 5       | Barbie         | 2023        | 114          | 5            | 2        |

---

# 🔗 2️⃣ Penggunaan JOIN

JOIN digunakan untuk menggabungkan data dari beberapa tabel yang memiliki relasi.

---

## 🟢 INNER JOIN

Menampilkan data yang memiliki relasi pada kedua tabel.

```sql
SELECT film.judul_film,
       sutradara.nama_sutradara,
       genre.nama_genre,
       film.tahun_rilis
FROM film
INNER JOIN sutradara
ON film.id_sutradara = sutradara.id_sutradara
INNER JOIN genre
ON film.id_genre = genre.id_genre;
```

INNER JOIN hanya menampilkan data yang **memiliki pasangan pada kedua tabel**.

---

## 🟡 LEFT JOIN

Menampilkan semua data dari tabel kiri meskipun tidak memiliki pasangan di tabel kanan.

```sql
SELECT sutradara.nama_sutradara, film.judul_film
FROM sutradara
LEFT JOIN film
ON sutradara.id_sutradara = film.id_sutradara;
```

LEFT JOIN memastikan **semua data sutradara tetap tampil**, walaupun belum memiliki film.

---

## 🔵 RIGHT JOIN

Menampilkan semua data dari tabel kanan meskipun tidak memiliki pasangan di tabel kiri.

```sql
SELECT genre.nama_genre, film.judul_film
FROM genre
RIGHT JOIN film
ON genre.id_genre = film.id_genre;
```

RIGHT JOIN memastikan **semua data film tetap tampil**, walaupun belum memiliki genre.

---

# 📊 Perbandingan Jenis JOIN

| Jenis JOIN | Keterangan                              |
| ---------- | --------------------------------------- |
| INNER JOIN | Hanya data yang memiliki relasi         |
| LEFT JOIN  | Semua data dari tabel kiri ditampilkan  |
| RIGHT JOIN | Semua data dari tabel kanan ditampilkan |

---

# 📌 Kesimpulan

Database **db_cinema** menunjukkan bagaimana sistem basis data relasional dapat digunakan untuk mengelola data film secara terstruktur.

Konsep yang digunakan dalam database ini meliputi:

* Pembuatan tabel relasional
* Penggunaan **primary key dan foreign key**
* Penggabungan data menggunakan **JOIN**
* Penyimpanan data film, sutradara, dan genre secara terstruktur
