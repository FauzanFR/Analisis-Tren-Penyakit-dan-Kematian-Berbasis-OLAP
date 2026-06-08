# Data Warehouse — Analisis Tren Penyakit dan Kematian Berbasis OLAP

Proyek akhir mata kuliah Data Warehouse, S1 Sains Data, Universitas Negeri Surabaya (2024-D).

## Anggota Kelompok

| Nama | NIM |
|------|-----|
| Fauzan Rafingudin | 24031554111 |
| Lucky Surya Revansyah | 24031554090 |
| M. Arief Afandi | 24031554181 |

---

## Deskripsi Proyek

Proyek ini merancang dan mengimplementasikan **Data Warehouse berbasis OLAP** untuk menganalisis tren penyakit dan kematian secara global. Data bersumber dari **World Health Organization (WHO)** dan **World Bank**, mencakup angka kematian, harapan hidup, populasi, penyebab kematian, dan indikator ekonomi dari berbagai negara pada periode **2017–2020**.

Dengan pendekatan star schema dan CUBE query di PostgreSQL, sistem ini memungkinkan analisis multidimensi yang fleksibel berdasarkan negara, tahun, income level, jenis kelamin, kelompok umur, dan penyebab kematian.

---

## Rumusan Masalah

Data kesehatan dari WHO dan World Bank tersedia secara publik namun masih tersebar dalam format tabel statistik yang besar dan belum terintegrasi. Database operasional konvensional tidak mampu mendukung kebutuhan analisis multidimensi secara cepat dan fleksibel. Diperlukan sistem Data Warehouse yang dapat mengintegrasikan, membersihkan, dan menyajikan data kesehatan global secara efisien untuk mendukung pengambilan keputusan.

---

## Dataset

| Sumber | Konten |
|--------|--------|
| World Health Organization (WHO) | Angka kematian, penyebab kematian, mortality rate |
| World Bank | Harapan hidup, populasi, GDP per capita, income level |

**Catatan dataset:**
- Terdapat missing values — ditangani dengan penghapusan baris atau imputasi median global (khusus GDP per capita)
- Distribusi tidak merata — negara middle income lebih dominan dibanding low income
- Outlier tidak ditangani secara khusus karena konteks data kesehatan; penghapusan outlier berisiko membias data

---

## Arsitektur Star Schema

```
                    DimAgeGroup
                         |
DimSex — FactDeath — DimCause
                         |
                    DimCountry
                         |
              DimIncomeLevel   DimYear
                         |
              FactCountryYear
```

### Tabel Fakta

| Tabel | Konten |
|-------|--------|
| `FactDeath` | Jumlah kematian (by negara, tahun, penyebab, usia, jenis kelamin) |
| `FactCountryYear` | Mortality rate, life expectancy, populasi, GDP per capita |

### Tabel Dimensi

| Tabel | Deskripsi |
|-------|-----------|
| `DimCountry` | Informasi negara dengan hierarki geografis |
| `DimYear` | Dimensi waktu (2017–2020) |
| `DimAgeGroup` | Kelompok umur |
| `DimCause` | Penyebab kematian |
| `DimSex` | Jenis kelamin |
| `DimIncomeLevel` | Klasifikasi tingkat pendapatan negara |

---

## Pipeline ETL

### Extract
Membaca file CSV dari sumber WHO dan World Bank, dikumpulkan per kategori tanpa modifikasi.

### Transform
- Cleaning missing values
- Standardisasi format nama negara dan tahun
- Seleksi dan penghapusan kolom yang tidak relevan
- Merge dataset berdasarkan atribut negara dan tahun

### Load
Import hasil transformasi ke PostgreSQL sesuai skema star schema yang telah dirancang. Performance benchmarking menunjukkan query join dan agregasi (SUM, AVG) berjalan dengan waktu eksekusi yang relatif cepat.

---

## Hasil Analisis OLAP

Analisis dilakukan menggunakan mekanisme **CUBE query** di PostgreSQL untuk eksplorasi multidimensi.

| # | Analisis | Temuan |
|---|----------|--------|
| 1 | Negara dengan kematian tertinggi (2017–2020) | Turks and Caicos Islands |
| 2 | Pengaruh income level terhadap kesehatan | Negara low income memiliki harapan hidup lebih rendah dan mortality rate lebih tinggi |
| 3 | Penyakit dengan kematian terbanyak | Penyakit sirkulasi & jantung mendominasi |
| 4 | Rata-rata harapan hidup vs angka kematian | Berkorelasi negatif — negara dengan mortality tinggi (Liberia, South Sudan) memiliki harapan hidup sangat rendah |
| 5 | Total kematian berdasarkan jenis kelamin | Kematian laki-laki lebih tinggi dibanding perempuan |
| 6 | Total kematian berdasarkan kelompok umur | Kelompok usia lanjut (75+) mendominasi angka kematian |
| 7 | Tren kematian 2017–2020 | Menunjukkan penurunan, terutama tajam di 2020 (kemungkinan artefak kelengkapan pelaporan) |

---

## Teknologi yang Digunakan

- **Python** — ETL pipeline, preprocessing, visualisasi
- **PostgreSQL** — Data Warehouse, CUBE query
- **Pandas** — Transformasi dan cleaning data
- **Matplotlib / Seaborn** — Visualisasi hasil OLAP

---

## Referensi

- https://doi.org/10.1016/j.eswa.2023.122481
- https://doi.org/10.1016/j.bdr.2021.100255
- https://doi.org/10.1016/j.jjimei.2025.100325
- https://doi.org/10.1108/JEIM-07-2024-0374