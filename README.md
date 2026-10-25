# Quantum Machine Learning untuk Cybersecurity
### Quantum Kernel (QSVC) vs SVM Klasik pada Deteksi Intrusi Jaringan, Dataset HQC-2026

## Tim
**Kelompok:** 

| Anggota Kelompok | NRP |
| :--- | :--- |
| **Muhammad Rakha Hananditya Rauf** | 5027241015 |
| **Abiyyu Raihan Putra Wikanto** | 5027241042 |
| **Aslam Ahmad Usman** | 5027241074 |

**Mata kuliah:** Quantum Computation for Cybersecurity, Institut Teknologi Sepuluh Nopember, 2026

Proyek kelompok mata kuliah **Quantum Computation for Cybersecurity**, Institut Teknologi Sepuluh Nopember (ITS).

Repositori ini berisi satu notebook Jupyter yang menguji secara jujur apakah *quantum kernel* membantu mendeteksi serangan jaringan pada dataset hibrida kuantum-klasik **HQC-2026**, lalu mengulang pengujian itu pada dataset kedua (**NSL-KDD**) sebagai pembanding.

> **Ringkasan temuan:** fitur QKD `qber` pada HQC-2026 bocor dari label (skor ≈ 1,00 bukan prestasi model). Pada fitur yang jujur, QSVC dan RBF-SVM **setara** setelah tuning yang adil, sehingga **tidak ditemukan bukti quantum advantage**. Hambatan utamanya adalah pembatasan 4 fitur (4 qubit), bukan jenis kernelnya.

---

## Daftar isi
1. [Pertanyaan penelitian](#pertanyaan-penelitian)
2. [Hasil utama](#hasil-utama)
3. [Isi notebook](#isi-notebook)
4. [Struktur repositori](#struktur-repositori)
5. [Dataset](#dataset)
6. [Cara menjalankan](#cara-menjalankan)
7. [Konfigurasi](#konfigurasi)
8. [Reproduksibilitas](#reproduksibilitas)
9. [Keterbatasan](#keterbatasan)
10. [Referensi dan atribusi](#referensi-dan-atribusi)
11. [Tim](#tim)

---

## Pertanyaan penelitian
| Kode | Pertanyaan |
|---|---|
| RQ1 | Apakah fitur QKD simulasi (`qber`, `sifted_key_rate`) benar-benar menambah kemampuan mendeteksi serangan, atau hanya membocorkan label? |
| RQ2 | Pada fitur yang jujur (tanpa kebocoran), apakah QSVC mengungguli SVM klasik setelah keduanya dituning dengan anggaran yang sama? |
| RQ3 | Bagaimana perilaku quantum kernel terhadap ukuran data, *shot noise*, dan biaya komputasi? |
| RQ4 | Apakah kesimpulan bertahan pada pembanding klasik yang kuat, split waktu, dan dataset lain tanpa fitur QKD? |
| RQ5 | Dari mana `qber` berasal secara fisik (BB84 + penyadap), dan seberapa tahan quantum kernel terhadap noise perangkat nyata? |

## Hasil utama
Angka berikut berasal dari eksekusi lengkap notebook (5 seed, 100 sampel test per kelas, simulator ideal). Seed dipatok, jadi hasil di lingkungan Anda seharusnya sama atau sangat mirip. Selalu cocokkan dengan sel **"Ringkasan otomatis"** di bagian 17 notebook.

| Temuan | Bukti |
|---|---|
| `qber` bocor dari label | AUC satu fitur = 1,000 (fitur jaringan terbaik 0,71); qber benign maks 0,032 vs serangan min 0,120; aturan "qber > 0,05" benar pada 100% data |
| QBER bukan properti aliran jaringan | Simulasi BB84: QBER ≈ 0,01 tanpa penyadap, ≈ 0,25 jika semua qubit disadap. QBER serangan di dataset (0,18) setara penyadapan ±68% qubit |
| QSVC ≈ RBF-SVM pada fitur jujur (tuned, 192 data/kelas) | A-PCA: 0,649 vs 0,631; A-ANOVA: 0,680 vs 0,671. Selisih ≤ ±0,03 dan selalu dalam variasi antar-seed |
| Pembatasan 4 fitur adalah hambatan utama | Random Forest 78 fitur: 0,759 pada data train yang sama |
| QSVC default tampak kalah hanya karena konfigurasi | Set ber-`qber`: QSVC default 0,81 sampai 0,95; setelah tuning 0,998 (RBF 1,00) |
| Dataset bersih (NSL-KDD) memberi kesimpulan sama | RBF-SVM, QSVC, dan Random Forest sekitar 0,78 sampai 0,80 pada 192 data/kelas |
| Split acak terlalu optimistis | Fitur jujur: recall serangan 64% (split acak) turun menjadi 1,4% (split waktu) |
| Noise menurunkan nilai kernel | K(x,x) ideal 1,0 turun ke 0,95 / 0,82 / 0,62 pada noise gerbang 0,005 / 0,02 / 0,05, dan 0,73 pada model noise `FakeManilaV2`. Efek ke akurasi belum terbukti pada skala kecil ini |
| VQC lebih lemah | ≈ 0,58 pada 24 dan 48 data/kelas (satu konfigurasi, 100 iterasi COBYLA) |

**Kalimat kesimpulan yang aman:** pada HQC-2026 v3 dan NSL-KDD, di simulator ideal dengan 4 qubit dan sampai 192 data per kelas, tidak ditemukan bukti quantum advantage.

## Isi notebook
Berkas: `QML_Cybersecurity_HQC2026_FINAL.ipynb`

| Bagian | Isi |
|---|---|
| 1 | Setup dan verifikasi berkas (SHA-256) |
| 2-3 | Data, audit mutu, audit *target leakage* (AUC per fitur, relasi deterministik, aturan satu baris) |
| 4 | **BB84 dan penyadap:** simulasi asal fisik QBER |
| 5 | Penyiapan fitur: set A / A-PCA / A-ANOVA / B / C / D, scaling sudut [0, π] |
| 6 | Mesin quantum kernel (statevector), **validasi terhadap `FidelityQuantumKernel` resmi**, biaya sirkuit |
| 7 | Eksperimen 1: ablation fitur, RBF-SVM vs QSVC, 5 seed |
| 8 | Eksperimen 2: tuning adil (80 vs 80 kombinasi) dan kurva belajar 12 sampai 192 data/kelas |
| 9 | Heatmap kernel dan *geometric difference* |
| 10 | Eksperimen 3: pengaruh *shot noise* |
| 11 | Eksperimen 4: noise perangkat (depolarizing dan fake backend IBM) |
| 12 | Eksperimen 5: VQC |
| 13 | Eksperimen 6: Random Forest skala penuh dan serangan yang sulit dideteksi |
| 14 | Eksperimen 7: split waktu |
| 15 | Eksperimen 8: dataset kedua NSL-KDD |
| 16 | Opsional: satu elemen kernel di IBM Quantum asli (mati secara default) |
| 17 | Ringkasan otomatis dan kesimpulan |

**Desain yang membuat perbandingan adil:**
- Split dulu, baru sampling dan scaling; scaler, PCA, dan seleksi fitur hanya dipelajari dari data train.
- Fitur dan split identik untuk semua model; tuning memakai validasi silang di data train saja.
- Anggaran tuning sama: 80 kombinasi untuk RBF-SVM dan 80 untuk QSVC.
- Metrik untuk data tidak seimbang: balanced accuracy, macro-F1, recall serangan, false-positive rate.
- Kernel kuantum dihitung dengan *statevector* (setara simulator ideal, ratusan kali lebih cepat), dan **dibuktikan sama** dengan Qiskit resmi (selisih kernel ≈ 5e-12, prediksi identik).

## Struktur repositori
```
.
├── README.md
├── QML_Cybersecurity_HQC2026_FINAL.ipynb     # notebook utama
├── requirements.txt
├── .gitignore
├── data/                                     # TIDAK di-commit (lihat bagian Dataset)
│   ├── SUPER_ENRICHED_hybrid_dataset_v3.parquet
│   ├── final_hybrid_cybersecurity_dataset.parquet
│   ├── KDDTrain+.txt                         # diunduh otomatis oleh notebook
│   └── KDDTest+.txt
└── results/                                  # CSV dan gambar hasil (boleh di-commit)
    ├── hasil_E1_ablation.csv
    ├── hasil_E2_tuning_kurva_belajar.csv
    ├── hasil_E3_shots.csv
    ├── hasil_E4_noise.csv
    ├── hasil_E4_vqc.csv
    ├── hasil_E5_rf_skala_penuh.csv
    ├── hasil_E6_split_waktu.csv
    ├── hasil_E8_nslkdd.csv
    └── gambar_*.png
```

Contoh `.gitignore`:
```
data/
*.parquet
KDD*.txt
.ipynb_checkpoints/
__pycache__/
```

## Dataset
Dataset **tidak disertakan** di repositori karena ukurannya besar. Unduh sendiri:

| Berkas | Sumber | Wajib? |
|---|---|---|
| `SUPER_ENRICHED_hybrid_dataset_v3.parquet` (±100 MB) | [Mendeley Data, DOI 10.17632/68rc9crgkj.1](https://data.mendeley.com/datasets/68rc9crgkj/1) (Download All, ZIP ±253 MB) | **Ya** |
| `final_hybrid_cybersecurity_dataset.parquet` (±94 MB) | Sama (ZIP yang sama) | Untuk split waktu (satu-satunya yang punya `timestamp`) |
| `HQC20_Super_Enriched_Dataset_v4.parquet` | Sama | Tidak dipakai |
| `KDDTrain+.txt`, `KDDTest+.txt` (NSL-KDD) | Diunduh **otomatis** oleh notebook dari GitHub; jika gagal, unggah manual ke folder data | Untuk Eksperimen 8 |

**Tentang Parquet:** format tabel berbasis kolom yang terkompresi, dibuka dengan `pd.read_parquet(...)` (bukan Excel). Notebook hanya membaca kolom yang dibutuhkan agar hemat memori.

**Catatan penting tentang HQC-2026:** halaman Mendeley (14,69 juta sampel seimbang, file OpenQASM) **tidak cocok** dengan isi arsip versi 1 (233.844 baris dengan 75,99% benign, tanpa file OpenQASM). Proyek ini memakai isi arsip yang sebenarnya. Fitur QKD berasal dari **simulasi** BB84, bukan dari perangkat kuantum.

## Cara menjalankan

### Opsi A: Google Colab (disarankan)
1. Unggah berkas Parquet ke Google Drive, folder `MyDrive/Quantum/` (atau ubah `DATA_DIR` di sel konfigurasi).
2. Buka notebook di Colab (File → Upload notebook).
3. Jalankan sel pertama (`pip install`), lalu **Runtime → Restart session** jika diminta.
4. **Runtime → Run all**. Waktu total sekitar 30 menit atau lebih (tergantung mesin). Hasil CSV dan gambar tersimpan otomatis ke folder Drive yang sama.

### Opsi B: Jupyter lokal / Kaggle
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    |    macOS/Linux: source .venv/bin/activate
pip install "qiskit-machine-learning==0.9.1" qiskit-aer qiskit-ibm-runtime \
            pandas pyarrow numpy scikit-learn matplotlib seaborn pylatexenc jupyter
export QML_DATA_DIR=/path/ke/data      # Windows PowerShell: $env:QML_DATA_DIR="C:\path\ke\data"
export QML_OUT_DIR=/path/ke/results    # opsional; default sama dengan QML_DATA_DIR
jupyter notebook QML_Cybersecurity_HQC2026_FINAL.ipynb
```
Di Kaggle: unggah berkas sebagai Dataset, nyalakan Internet, lalu set `DATA_DIR` ke `/kaggle/input/<nama-dataset>`.

## Konfigurasi
Semua pengaturan ada di satu sel ("KONFIGURASI") pada bagian 1 notebook:

| Variabel | Default | Fungsi |
|---|---|---|
| `DATA_DIR`, `OUT_DIR` | folder Drive / env var | Lokasi data dan hasil |
| `SEEDS` | `[0, 1, 2, 3, 4]` | Pengulangan eksperimen (minimal 5 disarankan) |
| `N_TEST` | `100` | Sampel test per kelas |
| `SIZES` | `[12, 24, 48, 96, 192]` | Ukuran train per kelas untuk kurva belajar |
| `RUN_VQC` / `RUN_NOISE` / `RUN_TIME_SPLIT` / `RUN_NSLKDD` | `True` | Hidup/matikan eksperimen tertentu |
| `RUN_IBM` | `False` | Perangkat IBM asli (butuh akun IBM Quantum) |

Untuk menjalankan lebih cepat, kecilkan `SEEDS` dan `SIZES` (misalnya `[0, 1, 2]` dan `[12, 48, 192]`).

## Reproduksibilitas
- Versi paket dipatok: `qiskit-machine-learning==0.9.1` (sesuai Bab 6). Notebook mencetak versi Python, numpy, pandas, scikit-learn, dan Qiskit di awal.
- Pengujian dilakukan dengan Python 3.12, Qiskit 2.6.1, scikit-learn 1.8.0, pandas 3.0.2. API Qiskit sering berubah; jika ada error import, cek versi dan dokumentasi resmi.
- Seed dipatok (`SEEDS`, `random_state`), dan notebook memverifikasi integritas berkas dengan **SHA-256**:

| Berkas | SHA-256 |
|---|---|
| `SUPER_ENRICHED_hybrid_dataset_v3.parquet` | `aa2aefc0e8586c56e315a36b6c9fa922f36b5c21c3390bf5c2ea2eeab9995ec1` |
| `HQC20_Super_Enriched_Dataset_v4.parquet` | `cdd7ce5d679955d8c19279b2a52a4f4562e2a35f551ab73043f34e0ec6aaab43` |
| `final_hybrid_cybersecurity_dataset.parquet` | `e1d49d73f4942bae20d8262dc03d38a466811e43375de42a7d1119ee6f9aedce` |

## Keterbatasan
1. Fitur QKD berasal dari **simulasi** BB84 dan `qber` tampak dibangkitkan dari label.
2. Subset kecil dan seimbang (sampai 192 per kelas), sedangkan data asli 76% benign. Skor pada subset seimbang tidak sama dengan performa pada lalu lintas asli.
3. Hasil utama memakai simulator ideal. Noise dipelajari pada eksperimen kecil (12 data per kelas, 4 qubit, model noise `FakeManilaV2`), bukan pada QPU nyata.
4. Ruang tuning terbatas (4 feature map, 5 bandwidth, 4 nilai C); VQC hanya satu konfigurasi.
5. Tidak ada data dictionary resmi untuk HQC-2026; satuan sebagian kolom tidak pasti.
6. Split waktu hanya satu skenario (70/30), dan jenis serangan Bot hanya muncul di test, jadi itu uji ketahanan terhadap pergeseran distribusi, bukan perbandingan setara.
7. Reduksi 78 fitur ke 4 (PCA/ANOVA) membuang informasi; teknik lain atau lebih banyak qubit bisa memberi hasil berbeda.
8. Eksperimen di IBM Quantum asli (bagian 16) tidak dijalankan dan kodenya belum diuji pada perangkat.

## Referensi dan atribusi
- Islam, I., Rabbi, I. R., Emon, T. H., Sahin Afridi, A., & Nur, F. (2026). *A Hybrid Quantum-Classical Network Intrusion Detection Dataset* (Version 1). Mendeley Data. https://doi.org/10.17632/68rc9crgkj.1 (CC BY 4.0). **Subset, transformasi fitur, dan pemetaan label dalam proyek ini adalah olahan kami.**
- Tavallaee, M., Bagheri, E., Lu, W., & Ghorbani, A. (2009). A detailed analysis of the KDD CUP 99 data set (NSL-KDD).
- Suryotrisongko, H. Materi kuliah Quantum Machine Learning ITS, Bab 4 (Metode Kernel Kuantum) dan Bab 6 (Dataset HQC-2026 untuk QML).
- Havlíček, V. et al. (2019). Supervised learning with quantum-enhanced feature spaces. *Nature* 567, 209-212.
- Huang, H.-Y. et al. (2021). Power of data in quantum machine learning. *Nature Communications* 12, 2631.
- Thanasilp, S., Wang, S., Cerezo, M., & Holmes, Z. (2024). Exponential concentration in quantum kernel methods. *Nature Communications* 15, 5200.
- Qiskit dan Qiskit Machine Learning, dokumentasi resmi.

Gunakan hanya untuk pendidikan dan penelitian defensif. Model yang akurat pada data benchmark belum tentu siap produksi; sistem deteksi intrusi nyata memerlukan validasi pada trafik organisasi, alarm palsu, dan jalur respons insiden.
