# 🚗 Dokumentasi Arsitektur & Analisis Basis Data: Sistem Manajemen Parkir (`db_parkir`)

Dokumentasi ini menyajikan pelacakan logistik interaktif pada `db_parkir`. Setiap tahap eksekusi telah diurutkan berdasarkan siklus hidup sebuah DBMS (Database Management System): mulai dari tahap pembuatan tabel, pemrosesan data, perhitungan dasar agregat, implementasi kueri tingkat lanjut antar tabel, hingga pemanfaatan SQL Views.


---

![mysql](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSXizOekeei5tf2Z8KG1_DI-hjwgLEttS-M9A&s)

---
## Kelompok 6

## 👥 Anggota Kelompok

1. [Asa Enggal Daviyana](https://www.linkedin.com/in/asa-enggal-daviyana-431ab63a5/)
2. [Muhammad Hubbi El Fairuz](https://www.linkedin.com/in/muhammad-hubbi-el-fairuz-3a3a633a5/)
3. Indira

---

## 🏗️ 1. Fase Pembuatan Database dan Tabel

Sistem manajemen parkir ini dibangun dengan tiga entitas utama yang dirangkai di dalam `db_parkir`. Berikut adalah skema strukturalnya:

### 🚙 1.1 `CREATE TABLE`: kendaraan

Berfungsi sebagai entitas master khusus kendaraan yang dioperasikan dalam sistem.

```sql
CREATE TABLE kendaraan (
    id_kendaraan int primary key auto_increment,
    nama_kendaraan varchar(20),
    nomor_plat varchar(10),
    jenis_kendaraan varchar(10),
    warna_kendaraan varchar(15)
);
```

**Struktur (DESCRIBE):**
| Field | Type | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id_kendaraan | int(11) | NO | PRI | NULL | auto_increment |
| nama_kendaraan | varchar(20) | YES | | NULL | |
| nomor_plat | varchar(10) | YES | | NULL | |
| jenis_kendaraan| varchar(10) | YES | | NULL | |
| warna_kendaraan| varchar(15) | YES | | NULL | |

### 👤 1.2 `CREATE TABLE`: user

Entitas pencatat dan pengelola pengguna atau pemilik kendaraan. Kolom _Foreign Key_ menautkan pengguna pada satu kendaraan valid atau disetel _NULL_.

```sql
CREATE TABLE user (
    id_user int primary key auto_increment,
    nama_user varchar(50),
    id_kendaraan int,
    foreign key (id_kendaraan) references kendaraan(id_kendaraan)
);
```

**Struktur (DESCRIBE):**
| Field | Type | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id_user | int(11) | NO | PRI | NULL | auto_increment |
| nama_user | varchar(50) | YES | | NULL | |
| id_kendaraan | int(11) | YES | MUL | NULL | |

### 🅿️ 1.3 `CREATE TABLE`: tempat_parkir

Mencatat alokasi area atau ruang parkir spesifik dan kendaraan yang menempatinya. Awalnya didefinisikan dengan kolom _jenis_slot_, namun sistem ini juga mengikutsertakan fase log modifikasi tabel (`ALTER`).

```sql
CREATE TABLE tempat_parkir (
    id_tepat_parkir int,
    kode_parkir varchar(10),
    jenis_slot varchar(10),
    id_kendaraan int,
    foreign key (id_kendaraan) references kendaraan(id_kendaraan)
);

-- Kolom 'jenis_slot' direstrukturisasi (Dihapus)
ALTER TABLE tempat_parkir DROP COLUMN jenis_slot;
```

**Struktur Akhir (DESCRIBE):**
| Field | Type | Null | Key | Default | Extra |
|--|--|--|--|--|--|
| id_tepat_parkir| int(11) | YES | | NULL | |
| kode_parkir | varchar(10) | YES | | NULL | |
| id_kendaraan | int(11) | YES | MUL | NULL | |

---

## 🛠️ 2. Fungsi Eksekusi Data (CRUD)

Fase ini menunjukan interaksi transaksional dasar pada arsitektur data tabel.

### 📥 2.1 CREATE Data (Eksekusi `INSERT`)

Sejumlah master data dimasukkan berturut-turut pada tabel.

- Mensuplai 10 armada unik (Mobil dan Motor) pada tabel `kendaraan`.
- Memasukkan profil 10 anggota kepada tabel `user`. Terdapat 3 _User_ yang tidak berafiliasi dengan id kendaraan manapun (`NULL`), yakni **Deni**, **Fajar**, dan **Intan**.
- Mendaftarkan blok parkir A1-C2 (10 _slot_) ke tabel `tempat_parkir`. Kendaraan beralokasi direfrensikan, dan lokasi sisanya (seperti A4, B3) menganggur (`NULL`).

### 📖 2.2 READ Data (`SELECT` dan Validasi)

Data terekam dapat diekstraksi. Penarikan data (Read) terkontrol memerlukan klausa pencarian tertentu.

**a. Menampilkan Pemilik Kendaraan Tidak Valid (Bukan Sama Dengan `NULL`):**
Adakalanya sistem hanya perlu mengidentifikasi pengguna mana yang secara harfiah tidak memiliki unit kendaraan.

```sql
SELECT * FROM user WHERE id_kendaraan IS NULL;
```

| id_user | nama_user | id_kendaraan |
| ------- | --------- | ------------ |
| 4       | Deni      | NULL         |
| 6       | Fajar     | NULL         |
| 9       | Intan     | NULL         |

**b. Menampilkan List Armada Urut Abjad Sesuai Jenis (WHERE & ORDER BY):**

```sql
-- Memilah Motor
SELECT * FROM kendaraan WHERE jenis_kendaraan = "Motor" ORDER BY nama_kendaraan ASC;

-- Memilah Mobil
SELECT * FROM kendaraan WHERE jenis_kendaraan = "Mobil" ORDER BY nama_kendaraan ASC;
```

**Hasil Filter Tabel Motor:**
| id_kendaraan | nama_kendaraan | nomor_plat | jenis_kendaraan | warna_kendaraan |
|--|--|--|--|--|
| 8 | Aerox | B8901HH | Motor | Silver |
| 3 | Beat | B3456CC | Motor | Merah |
| 4 | Nmax | B4567DD | Motor | Biru |
| 10 | Supra | B0123JJ | Motor | Merah |
| 6 | Vario | B6789FF | Motor | Hitam |

**Hasil Filter Tabel Mobil:**
| id_kendaraan | nama_kendaraan | nomor_plat | jenis_kendaraan | warna_kendaraan |
|--|--|--|--|--|
| 1 | Avanza | B1234AA | Mobil | Hitam |
| 7 | Brio | B7890GG | Mobil | Kuning |
| 2 | Civic | B2345BB | Mobil | Putih |
| 5 | Fortuner | B5678EE | Mobil | Abu |
| 9 | Xpander | B9012II | Mobil | Putih |

### 🔄 2.3 UPDATE Data

Penyegaran data saat kendaraan beranjak dari slot dan status tempat parkir C1 dialihkan jadi kosong (Available).

```sql
UPDATE tempat_parkir SET id_kendaraan = null WHERE kode_parkir = "C1";

SELECT * FROM tempat_parkir;
```

| id_tepat_parkir | kode_parkir | id_kendaraan |
| --------------- | ----------- | ------------ |
| 1               | A1          | 1            |
| 2               | A2          | 2            |
| 3               | A3          | 3            |
| 4               | A4          | NULL         |
| 5               | B1          | 5            |
| 6               | B2          | 6            |
| 7               | B3          | NULL         |
| 8               | B4          | 8            |
| 9               | C1          | NULL         |
| 10              | C2          | NULL         |

### 🗑️ 2.4 DELETE Data

Langkah ini mempermanenkan penghapusan unit dengan memusnahkan **Xpander** yang memiliki _ID 9_ di dalam memori database `kendaraan`.

```sql
DELETE FROM kendaraan WHERE id_kendaraan = 9;

SELECT * FROM kendaraan;
```

| id_kendaraan | nama_kendaraan | nomor_plat | jenis_kendaraan | warna_kendaraan |
| ------------ | -------------- | ---------- | --------------- | --------------- |
| 1            | Avanza         | B1234AA    | Mobil           | Hitam           |
| 2            | Civic          | B2345BB    | Mobil           | Putih           |
| 3            | Beat           | B3456CC    | Motor           | Merah           |
| 4            | Nmax           | B4567DD    | Motor           | Biru            |
| 5            | Fortuner       | B5678EE    | Mobil           | Abu             |
| 6            | Vario          | B6789FF    | Motor           | Hitam           |
| 7            | Brio           | B7890GG    | Mobil           | Kuning          |
| 8            | Aerox          | B8901HH    | Motor           | Silver          |
| 10           | Supra          | B0123JJ    | Motor           | Merah           |

_(Tabel di atas mengkonfirmasi hanya tersisa 9 inventaris baris data)._

---

## 🧮 3. Agregasi: Menghitung Frekuensi Data (COUNT)

Perhitungan kalkulatif memetakan seberapa banyak baris profil yang teregistrasi sah memakai utilitas parameter nilai valid (`IS NOT NULL`).

**Menghitung Total _User_ Yang Mendaftarkan Kendaraannya Secara Legal:**

```sql
SELECT count(*) FROM user WHERE id_kendaraan IS NOT NULL;
```

| count(\*) |
| --------- |
| 7         |

> Jumlah ini persis mencerminkan populasi entitas pengguna aktif yang mana berpadanan langsung dengan jumlah sisa pemilik (Idensi: Budi, Citra dkk).

---

## 🔗 4. Operasi Relasional (Pemetaan JOIN)

Ekspansi terpenting saat dua tabel referensial saling melempar informasi terkait log _id kendaraan_.

### 🟢 4.1 Posisi Data Eksak: `INNER JOIN`

Bersifat mengabaikan _baris tak sah / tak cocok_, menampilkan murni apa yang saling memvalidasi relasinya.

**a. Daftar Pemilik Sah dan Identitas Penuh Kendaraannya:**

```sql
SELECT user.nama_user, kendaraan.nama_kendaraan
FROM user
INNER JOIN kendaraan ON user.id_kendaraan = kendaraan.id_kendaraan;
```

| nama_user | nama_kendaraan |
| --------- | -------------- |
| Andi      | Avanza         |
| Budi      | Civic          |
| Citra     | Beat           |
| Eka       | Fortuner       |
| Gilang    | Brio           |
| Hadi      | Aerox          |
| Joko      | Supra          |

**b. Posisi Lacak Slot Parkir:**

```sql
SELECT kendaraan.nama_kendaraan, tempat_parkir.kode_parkir
FROM tempat_parkir
INNER JOIN kendaraan ON tempat_parkir.id_kendaraan = kendaraan.id_kendaraan;
```

| nama_kendaraan | kode_parkir |
| -------------- | ----------- |
| Avanza         | A1          |
| Civic          | A2          |
| Beat           | A3          |
| Fortuner       | B1          |
| Vario          | B2          |
| Aerox          | B4          |
| Xpander        | C1          |

_(Ditampilkan 7 Slot Terisi)_

### 🟢 4.2 Posisi Data Eksplisit (Dengan Null): `LEFT JOIN` & `RIGHT JOIN`

Metode mempertahankan rekaman penuh sumber meski bagian _Referential Integrity_ ada yang bernilai kosong (`NULL`).

**a. Menampilkan Daftar Lengkap _Semua_ Profil User (`LEFT JOIN`):**

```sql
SELECT user.nama_user, kendaraan.nama_kendaraan
FROM user
LEFT JOIN kendaraan ON user.id_kendaraan = kendaraan.id_kendaraan;
```

| nama_user | nama_kendaraan |
| --------- | -------------- |
| Andi      | Avanza         |
| Budi      | Civic          |
| Citra     | Beat           |
| Deni      | NULL           |
| Eka       | Fortuner       |
| Fajar     | NULL           |
| Gilang    | Brio           |
| Hadi      | Aerox          |
| Intan     | NULL           |
| Joko      | Supra          |

**b. Menampilkan Inventaris Lengkap _Semua_ Kendaraan (`RIGHT JOIN`):**

```sql
SELECT kendaraan.nama_kendaraan, tempat_parkir.kode_parkir
FROM tempat_parkir
RIGHT JOIN kendaraan ON tempat_parkir.id_kendaraan = kendaraan.id_kendaraan;
```

| nama_kendaraan | kode_parkir |
| -------------- | ----------- |
| Avanza         | A1          |
| Civic          | A2          |
| Beat           | A3          |
| Nmax           | NULL        |
| Fortuner       | B1          |
| Vario          | B2          |
| Brio           | NULL        |
| Aerox          | B4          |
| Xpander        | C1          |
| Supra          | NULL        |

**c. Pemfilteran Kategori Berantai via JOIN (Cari "Motor" yang Sedang Berkendara / Tidak Berparkir):**

```sql
SELECT kendaraan.nama_kendaraan, tempat_parkir.kode_parkir
FROM tempat_parkir
RIGHT JOIN kendaraan ON tempat_parkir.id_kendaraan = kendaraan.id_kendaraan
WHERE tempat_parkir.kode_parkir IS NULL AND kendaraan.jenis_kendaraan = "Motor";
```

| nama_kendaraan | kode_parkir |
| -------------- | ----------- |
| Nmax           | NULL        |
| Supra          | NULL        |

> Operasi ini mensintesis relasi yang sangat cerdas: Mensyaratkan sistem mencari di sisi _kanan_ (semua Motor), lalu dicari yang blok parkirnya tidak ditemukan (`IS NULL`).

---

## 👁️ 5. Abstraksi Data (Implementasi VIEW)

`VIEW` diimplementasikan untuk menyederhanakan kueri SQL yang kompleks dengan menyimpannya sebagai "tabel virtual". View sangat praktis dalam mengamankan data maupun mempercepat analisis karena cukup dipanggil menggunakan perintah `SELECT` biasa tanpa melampirkan kueri `JOIN` panjangnya tiap kali.

### 🟡 5.1 Menyatukan Seluruh Entitas (`view_parkir_lengkap`)

View ini digabungkan menggunakan sintaks `LEFT JOIN` berganda untuk menarik representasi penuh dan detail menyeluruh dari ketiga tabel master ke satu tempat.

```sql
CREATE VIEW view_parkir_lengkap AS
SELECT
    user.nama_user,
    kendaraan.nama_kendaraan,
    kendaraan.nomor_plat,
    kendaraan.jenis_kendaraan,
    tempat_parkir.kode_parkir
FROM user
LEFT JOIN kendaraan ON user.id_kendaraan = kendaraan.id_kendaraan
LEFT JOIN tempat_parkir ON kendaraan.id_kendaraan = tempat_parkir.id_kendaraan;
```

**Tabel Hasil Panggilan `SELECT * FROM view_parkir_lengkap;`:**
| nama_user | nama_kendaraan | nomor_plat | jenis_kendaraan | kode_parkir |
|--|--|--|--|--|
| Andi | Avanza | B1234AA | Mobil | A1 |
| Budi | Civic | B2345BB | Mobil | A2 |
| Citra | Beat | B3456CC | Motor | A3 |
| Deni | NULL | NULL | NULL | NULL |
| Eka | Fortuner | B5678EE | Mobil | B1 |
| Fajar | NULL | NULL | NULL | NULL |
| Gilang | Brio | B7890GG | Mobil | NULL |
| Hadi | Aerox | B8901HH | Motor | B4 |
| Intan | NULL | NULL | NULL | NULL |
| Joko | Supra | B0123JJ | Motor | NULL |

> **Analisis:** _view_parkir_lengkap_ mendemostrasikan dengan gamblang rekap data dari sepuluh row user. Sangat praktis diamati: _Deni_, _Fajar_, _Intan_ bernilai NULL mutlak karena tak memilki kendaraan sejak tabel user awal. Sementara user bernama _Gilang_ dan _Joko_ tercatut kendaraannya namun mereka tidak parkir / status kode_parkirnya NULL.

### 🟡 5.2 Mengisolasi Subjek Tanpa Kendaraan (`view_user_tanpa_kendaraan`)

View ini berperforma sebagai pengayak (filer) instan. Tujuannya membidik `id_user` dan `nama_user` manakala nilai ID kendaraannya kosong.

```sql
CREATE VIEW view_user_tanpa_kendaraan AS
SELECT
    id_user,
    nama_user
FROM user
WHERE id_kendaraan IS NULL;
```

**Tabel Hasil Panggilan `SELECT * FROM view_user_tanpa_kendaraan;`:**
| id_user | nama_user |
|--|--|
| 4 | Deni |
| 6 | Fajar |
| 9 | Intan |

> **Analisis:** Sangat berguna bagi aplikasi apabila manajemen parkiran perlu mengingatkan orang-orang yang status parkir / akunnya tidak lengkap secara terfokus. Modul pelapor tak perlu mengingat script `WHERE` kompleks, cukup menyeleksi dari view ini saja.

### 🟡 5.3 Memantau Okupansi Slot Secara Terfokus (`view_kendaraan_parkir`)

View penarikan berbasis relasi kaku atau absolut (`JOIN`). Tujuannya adalah secara eksklusif berfokus pada slot-slot spesifik yang saat ini _sedang diduduki_ benda bergerak di area parkir.

```sql
CREATE VIEW view_kendaraan_parkir AS
SELECT
    kendaraan.nomor_plat,
    kendaraan.jenis_kendaraan,
    tempat_parkir.kode_parkir
FROM kendaraan
JOIN tempat_parkir ON kendaraan.id_kendaraan = tempat_parkir.id_kendaraan;
```

**Tabel Hasil Panggilan `SELECT * FROM view_kendaraan_parkir;`:**
| nomor_plat | jenis_kendaraan | kode_parkir |
|--|--|--|
| B1234AA | Mobil | A1 |
| B2345BB | Mobil | A2 |
| B3456CC | Motor | A3 |
| B5678EE | Mobil | B1 |
| B6789FF | Motor | B2 |
| B8901HH | Motor | B4 |

> **Analisis:** _view_kendaraan_parkir_ meminimalisir detail data privasi profil pemilik (nama_user) di dalamnya. Fokus view ini diarahkan utuh hanya pada properti plat nomor, tipe, serta presisi posisi lokasi parkirnya di area (A1-B4) yang bernilai tidak null.

---
