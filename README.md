# 🏦 Analisis Risiko Kredit (Credit Risk): Solusi End-to-End Business Intelligence

![Dashboard Preview]<img width="1335" height="730" alt="image" src="https://github.com/user-attachments/assets/cc02c4ac-bf75-4aa1-a564-efbabd99eaa2" />

> *Dashboard Manajemen Risiko Eksekutif untuk meminimalisir Non-Performing Loans (NPL) melalui segmentasi nasabah berbasis data.*

## 📌 Ringkasan Proyek (Executive Summary)

**Konteks Bisnis:**
Dalam industri pembiayaan konsumen, kemampuan memprediksi kapasitas bayar nasabah adalah kunci keberlanjutan bisnis. Memberi pinjaman ke orang yang salah menyebabkan kerugian finansial (*Non-Performing Loan*), sementara menolak nasabah potensial menyebabkan hilangnya peluang keuntungan (*Opportunity Loss*).

**Tujuan:**
Membangun **pipeline data otomatis** dan **dashboard interaktif** yang memungkinkan Tim Manajemen Risiko untuk:
1.  Mengidentifikasi segmen demografi berisiko tinggi secara akurat.
2.  Mendeteksi pola "risiko tersembunyi" (seperti nasabah yang *Over-leveraged*).
3.  Melakukan simulasi strategi kredit melalui fitur *cross-filtering*.

**Tools yang Digunakan:**
* **SQL (PostgreSQL):** Data Cleaning, Complex Joins, CTEs, Window Functions.
* **Tableau Public:** Visualisasi Data Interaktif & Dashboard Actions.
* **DBeaver:** Database Management.

---

## 📂 Sumber Data (Dataset)

Proyek ini menggunakan dataset **Home Credit Default Risk** yang mensimulasikan data perilaku kredit nasabah di dunia nyata.
* **Sumber:** [Kaggle - Home Credit Default Risk](https://www.kaggle.com/c/home-credit-default-risk/data)
* **Tabel yang Digunakan:**
    * `application_train`: Data profil nasabah saat ini.
    * `previous_application`: Riwayat pengajuan kredit nasabah di internal Home Credit.
    * `bureau`: Riwayat kredit nasabah di lembaga keuangan lain (data eksternal).

---

## 🔍 Temuan Investigasi (Exploratory Data Analysis)

Sebelum membangun dashboard, saya melakukan 6 tahap analisis mendalam menggunakan SQL untuk memvalidasi hipotesis risiko. Berikut adalah rangkuman temuan dari file `02_eda_investigation.sql`:

### 1. Menentukan Baseline (Tolok Ukur Nasional)
Langkah pertama adalah mencari angka rata-rata gagal bayar sebagai acuan (*Threshold*).
* **Temuan:** Rata-rata Bad Rate Nasional adalah **~8.07%**.
* **Implikasi:** Segmen apa pun yang memiliki Bad Rate di atas 8.07% akan dikategorikan sebagai **"High Risk"** (Zona Merah).

### 2. Risiko Berdasarkan Pekerjaan (Income Type)
* **Hipotesis:** Apakah status pekerjaan mempengaruhi risiko?
* **Temuan:**
    * **Working (Buruh/Karyawan):** Risiko Tinggi (**9.59%**).
    * **Pensioner (Pensiunan):** Risiko Sangat Rendah (**5.39%**).
    * *Insight:* Pensiunan cenderung lebih disiplin membayar dibanding karyawan aktif.

### 3. Risiko Berdasarkan Umur (Age Group)
* **Hipotesis:** Nasabah muda kurang matang secara finansial.
* **Bukti Data:** Terdapat korelasi negatif sempurna.
    * **< 25 Tahun:** Risiko melambung ke **12.3%** (Sangat Berisiko).
    * **> 55 Tahun:** Risiko turun ke **5.1%**.
* **Rekomendasi Awal:** Perketat *approval* untuk nasabah di bawah 25 tahun.

### 4. Analisis Riwayat Penolakan (Historical Behavior)
* **Hipotesis:** Nasabah yang pernah ditolak sebelumnya akan macet lagi.
* **Teknis SQL:** Menggunakan *Common Table Expression (CTE)* untuk mengagregasi tabel `previous_application`.
* **Temuan:** Nasabah yang statusnya **"Pernah Ditolak (Refused)"** memiliki risiko **10.3%**, jauh lebih tinggi dibanding nasabah bersih (6.9%).

### 5. Analisis Kemampuan Bayar (Affordability Ratio)
* **Hipotesis:** Semakin besar cicilan dibanding gaji (*Annuity/Income*), semakin besar risiko macet.
* **Temuan Anomali:**
    * Rasio Cicilan **<10%** adalah yang paling aman.
    * Namun, risiko tertinggi justru ada di kelas menengah (Rasio **20-30%**), bukan di rasio terbesar (>40%).
    * *Analisis:* Nasabah dengan rasio >40% mungkin merupakan segmen *High Net Worth* yang memiliki sumber dana lain, sehingga tetap lancar bayar.

### 6. Analisis Utang Luar (Bureau / External Debt)
* **Hipotesis:** Fenomena "Gali Lubang Tutup Lubang" (*Over-leveraged*).
* **Temuan Kritis:** Nasabah dengan **5+ Utang Aktif** di bank lain memiliki risiko meledak hingga **12.05%**. Ini adalah indikator bahaya yang sangat kuat.

---

## 📊 Strategi Dashboard: Cross-Analysis & Interactivity

Alih-alih menyajikan data statis, Dashboard ini dirancang dengan fitur **Action Filters** untuk memungkinkan *Risk Manager* melakukan validasi hipotesis secara mandiri.

**Skenario Penggunaan (Use Case):**

*"Apakah benar semua Anak Muda (<25 Tahun) itu berisiko tinggi dan harus ditolak?"*

Dengan Dashboard ini, user dapat melakukan **Drill-Down Analysis**:
1.  **Klik** batang grafik umur *"< 25 Years"*.
2.  Secara otomatis, KPI **Total Applicants**, **Bad Rate**, dan **Total Defaulters** akan berubah mengkalkulasi ulang khusus untuk segmen tersebut.
3.  User kemudian dapat melihat grafik *Affordability* dan *Bureau Debt* di bagian bawah yang sudah terfilter.

**Tujuan:**
Fitur ini memungkinkan penemuan **"Micro-Segments"** potensial. Misalnya, user dapat membuktikan apakah Anak Muda yang *Tidak Punya Utang Luar* sebenarnya memiliki profil risiko yang aman (Hijau) sehingga layak diberi pinjaman ("Hidden Gem").

👉 **[Lihat Dashboard Live di Tableau Public](MASUKKAN_LINK_TABLEAU_ANDA_DISINI)**

---

## 📂 Struktur Repository

* `/sql_scripts`: Berisi kode query SQL lengkap (PostgreSQL).
    * `01_data_prep.sql`: Script pembersihan dan pembuatan View.
    * `02_eda_investigation.sql`: 6 Query analisis utama (Baseline, Demografi, Behavior, Financial).
    * `03_final_mart.sql`: Script pembuatan tabel final denormalisasi untuk BI.
* `/assets`: Berisi screenshot bukti hasil query dan tampilan dashboard.

---

## 📝 Penulis
**[Nama Lengkap Anda]**
Data Analyst Enthusiast | SQL & Tableau Specialist
[Link LinkedIn Anda] | [Email Anda]
