# Dampak COVID-19 terhadap Pergeseran Struktural Sistem Pembayaran Indonesia

Analisis structural break dan difference-in-differences terhadap data resmi Bank Indonesia untuk menguji apakah COVID-19 menyebabkan pergeseran signifikan dari pembayaran kartu (ATM/Debit) ke uang elektronik — dan implikasinya terhadap risiko konsentrasi dana pada penerbit non-bank.

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![statsmodels](https://img.shields.io/badge/statsmodels-333333?style=flat-square)
![Power Query](https://img.shields.io/badge/Power_Query-217346?style=flat-square&logo=microsoftexcel&logoColor=white)
![Status](https://img.shields.io/badge/status-completed-2ea44f?style=flat-square)

<p align="center">
  <a href="analisis_dampak_covid_sistem_pembayaran.html"><strong>Buka Notebook (HTML) →</strong></a>
</p>

## Contents
- [Business Problem](#business-problem)
- [Data](#data)
- [Data Preparation — Power Query](#data-preparation--power-query)
- [Methodology — Python](#methodology--python)
- [Key Findings](#key-findings)
- [Hipotesis Awal yang Ditolak](#hipotesis-awal-yang-ditolak)
- [Limitations](#limitations)

## Business Problem
Apakah COVID-19 (April 2020) menyebabkan perubahan struktural yang signifikan secara statistik pada dominasi uang elektronik atas ATM/Debit di Indonesia — dan bagaimana implikasinya terhadap risiko konsentrasi dana float pada penerbit non-bank?

## Data
- **Sumber:** Bank Indonesia — Statistik Sistem Pembayaran dan Infrastruktur Pasar Keuangan (SPIP) & Statistik Sistem Keuangan Indonesia (SSKI), publikasi Agustus 2026, `bi.go.id/id/statistik/ekonomi-keuangan/`
- **Tabel SPIP:** 5e (Uang Elektronik), 5a (ATM dan ATM+Debet), 5f (Infrastruktur — jumlah merchant), 2 (Media Pembayaran — M1)
- **Tabel SSKI:** 19 (Indikator Keuangan Inklusif — termasuk breakdown regional), 20 (Indikator UMKM)
- **Cakupan:** bulanan, Januari 2009 – Juli 2026 (211 observasi)

## Data Preparation — Power Query
1. `Get Data → From File → From Workbook` → pilih sheet `5e`/`5a` (cari berdasarkan judul di Table of Contents, bukan nomor sheet — lihat catatan di atas)
2. `Remove Top Rows` (6 baris) → `Use First Row as Headers`
3. Hapus kolom "tahunan" (duplikat nilai Desember) di setiap blok 13-kolom
4. `Unpivot Columns` pada seluruh kolom bulan → hasil: kolom `Attribute` (bulan) + `Value` (angka)
5. Bikin kolom tanggal berurutan dari index baris (data berurutan kronologis tanpa lompatan)
6. `Merge Queries` antara e-money dan ATM/Debit berdasarkan tanggal
7. Export ke CSV untuk tahap Python

## Methodology — Python
- **Deteksi breakpoint otomatis** (`ruptures`, algoritma PELT) — tanpa diberi tahu tanggal kebijakan, untuk uji independen
- **Segmented regression / interrupted time series** (`statsmodels` OLS) dengan variabel dummy interaksi, menguji level jump dan slope change di April 2020
- **Difference-in-differences** — membandingkan pertumbuhan e-money vs ATM/Debit (kelompok kontrol) 12 bulan sebelum/sesudah
- **Normalisasi terhadap M1** (uang beredar) — memastikan temuan bukan peninggalan inflasi/ekspansi moneter
- **Regresi tren linier** pada pangsa dana float non-bank + proyeksi

## Key Findings
- Uang elektronik melampaui ATM/Debit dalam nilai transaksi sejak ~April 2020; pada Juli 2026 nilainya **2,64x lipat** dari ATM/Debit
- Segmented regression: level jump signifikan di April 2020 (koefisien +1,06, p<0,001), R²=0,943 — tapi **algoritma deteksi breakpoint otomatis tidak menempatkan April 2020 sebagai titik patah paling dominan** (titik lain: 2012, 2017, 2019, 2024 juga signifikan) — kesimpulan jujur: COVID adalah akselerator tren struktural jangka panjang, bukan pencipta tren dari nol
- Diff-in-diff: e-money +29,1% vs ATM/Debit **-17,4%** (12 bulan sebelum/sesudah April 2020) — efek bersih +46,5 poin persentase
- E-money sebagai % M1 naik dari 0,01% (2012) ke 1,85% (2026) — ~185x lipat, membuktikan pergeseran perilaku riil, bukan sekadar inflasi
- Pangsa dana float pada penerbit non-bank naik dari 16,2% (2015) ke 72,5% (2026), tren linier kuat (R²=0,885), berpotensi tembus 90% sekitar 2029 jika berlanjut

## Perluasan Analisis — Inklusi Keuangan & UMKM (Tabel 19 & 20 SSKI)
- **Ketimpangan regional tajam:** jumlah rekening e-money registered pada Agen LKD per 1.000 penduduk dewasa di Jawa (140,9) adalah **48,3x lipat** dari Maluku & Papua (2,9) — pertumbuhan e-money nasional sangat tidak merata secara geografis
- **Korelasi dengan akses kredit UMKM diuji dan terbukti semu (spurious):** korelasi level mentah antara nilai transaksi e-money dan pangsa kredit UMKM terlihat kuat (-0,87), tapi setelah diuji dengan first-difference (mengontrol tren bersama), korelasinya jatuh ke -0,10 — **tidak ada bukti nyata bahwa e-money memengaruhi akses kredit UMKM**. Ini dilaporkan apa adanya sebagai contoh kehati-hatian terhadap korelasi semu pada dua deret waktu yang sama-sama bertren

## Hipotesis Awal yang Ditolak
Hipotesis awal project ini adalah kenaikan limit transaksi QRIS (2021-2022) mempercepat pertumbuhan e-money. Data membuktikan **sebaliknya** — ATM/Debit tumbuh lebih cepat dari e-money pada periode itu, kemungkinan karena efek dasar (base effect) pemulihan ekonomi pasca-PPKM.

## Limitations
Data QRIS granular tidak tersedia gratis dari BI — proxy yang dipakai adalah uang elektronik secara keseluruhan. Analisis bersifat kuasi-eksperimental (structural break + diff-in-diff pada data agregat nasional bulanan), bukan eksperimen acak murni, tidak bisa 100% menyingkirkan faktor perancu lain yang bertepatan dengan April 2020.

---

<sub>**Muhammad Yahya Ayyasy** — [LinkedIn](https://linkedin.com/in/muhammadayyass) · [muhammadayyas22@gmail.com](mailto:muhammadayyas22@gmail.com)</sub>
