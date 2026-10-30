# STEM-Err-ID

**Evaluasi dan Mitigasi Kesalahan Large Language Model Skala Kecil pada Penyelesaian Soal Science Berbahasa Indonesia**

Karya untuk DIMAS-TI 2026, kategori Penambangan Data.

> **Status:** Selesai. Repositori ini memuat kode, hasil audit data, dan hasil eksperimen dari makalah final.

## Latar Belakang

Adopsi kecerdasan buatan dalam pendidikan bergerak lebih cepat daripada pemahaman tentang batas kepercayaannya, terutama untuk konteks kurikulum nasional. Proyek ini bertujuan mengevaluasi kinerja model bahasa berskala kecil (Small Language Models / SLM dengan parameter ≤4B) dalam menyelesaikan soal sains berbahasa Indonesia. Kami **memeriksa** kinerja model secara empiris: seberapa akurat mereka, di soal mana mereka rentan salah, mengapa mereka salah (termasuk fenomena *overconfidence* dan bias opsi), serta menguji sejauh mana teknik mitigasi *Self-Consistency* (majority voting) mampu memperbaiki kesalahan tersebut.

## Tujuan Penelitian

1. Menyusun subset soal sains (IPA, Fisika, Kimia, dan Biologi) dari IndoMMLU yang diaudit dan dibersihkan sebagai sumber data yang dapat direplikasi.
2. Mengevaluasi akurasi lima SLM secara *zero-shot* dengan prosedur penilaian seragam.
3. Mengidentifikasi pola kesalahan prediksi model (muatan angka, panjang kalimat, kalibrasi *confidence*, dan bias opsi jawaban).
4. Menguji efektivitas *Self-Consistency* (*majority voting*) dalam memperbaiki soal yang dijawab salah pada tahap *zero-shot*, serta membandingkannya antar model.

## Data

- **Subset Sains IndoMMLU**: Setelah melalui proses audit dan pembersihan, diperoleh **2.925 soal sains bersih** yang mencakup mata pelajaran IPA, Fisika, Kimia, dan Biologi dari jenjang SD hingga seleksi masuk PTN.
- Sumber Asli: [indolem/IndoMMLU](https://huggingface.co/datasets/indolem/IndoMMLU) dan [fajri91/IndoMMLU](https://github.com/fajri91/IndoMMLU).
- Lisensi: CC BY-NC-SA 4.0 (atribusi wajib dan non-komersial). 
- Data **tidak** disertakan langsung di repo ini untuk mematuhi lisensi. Silakan jalankan *script* pra-pemrosesan kami untuk menghasilkan subset data yang sama dari sumber aslinya.

## Model

Model terbuka ≤4B parameter dari keluarga berbeda diuji secara *zero-shot* dan dievaluasi lebih lanjut menggunakan *Self-Consistency*.

| Model | Parameter | Pendekatan Evaluasi |
|---|---|---|
| `Qwen2.5-3B` | ~3B | Zero-shot & Self-Consistency |
| `Phi-3.5-mini-instruct` | ~3.8B | Zero-shot & Self-Consistency |
| `StableLM-Zephyr-3B` | ~3B | Zero-shot |
| `SmolLM2-1.7B-Instruct` | ~1.7B | Zero-shot |
| `TinyLlama-1.1B-Chat-v1.0`| ~1.1B | Zero-shot & Self-Consistency |

## Metode

1. **Audit & Preprocessing Data:** Pembersihan dataset IndoMMLU dari soal duplikat, opsi kosong, dan kunci jawaban di luar rentang opsi.
2. **Evaluasi Awal (Zero-shot):** Menggunakan sampel uji coba (*pilot sample*) sebanyak 100 soal unik. Model menilai probabilitas huruf jawaban (A–E).
3. **Analisis Kesalahan:** Mengelompokkan tingkat kesalahan berdasarkan karakteristik soal (muatan kuantitatif/angka vs konseptual, panjang kalimat), serta menganalisis kalibrasi *confidence* dan bias opsi jawaban.
4. **Mitigasi (Self-Consistency):** Membangkitkan 5 jawaban independen (*temperature* = 0.7) untuk soal yang salah, lalu menggunakan *majority voting* untuk menentukan jawaban akhir.

## Hasil

### 1. Performa Awal (Zero-Shot)
Pada pengujian 100 soal *pilot*:
* **Qwen2.5-3B**: 50,0%
* **Phi-3.5-mini**: 38,0%
* **SmolLM2-1.7B**: 28,0% (Memiliki bias condong memilih opsi C sebesar 52,0%)
* **StableLM-3B**: 26,0%
* **TinyLlama-1.1B**: 22,0% (Setara tebakan acak)

*Catatan: Kesalahan prediksi pada kelima SLM secara konsisten dipicu oleh muatan angka pada soal dan kompleksitas kalimat (>15 kata). Model Phi-3.5-mini juga menunjukkan fenomena overconfidence (skor keyakinan 81,7% saat menjawab salah).*

### 2. Efektivitas Mitigasi (Self-Consistency)
Pengujian pada soal yang dijawab salah di tahap evaluasi awal:

| Model | Akurasi Awal | Akurasi Pasca-Mitigasi | Efektivitas Perbaikan |
|---|---|---|---|
| `Qwen2.5-3B` | 50,0% | 51,0% | **2,00%** (1 dari 50 salah diperbaiki) |
| `Phi-3.5-mini` | 38,0% | 46,0% | **12,90%** (8 dari 62 salah diperbaiki)|
| `TinyLlama-1.1B` | 22,0% | 27,0% | **6,41%** (5 dari 78 salah diperbaiki) |

**Kesimpulan Utama:** *Self-Consistency* (pemanfaatan *majority voting*) sangat efektif meredam kecerobohan penalaran (*reasoning noise*) pada model kelas menengah seperti Phi-3.5-mini, namun gagal mengoreksi ketidaktahuan konseptual (*knowledge error*) yang konsisten pada model terkuat seperti Qwen2.5-3B.

## Struktur Repo

## Struktur Repo

```text
├── arsip_pilot/
│   ├── hasil_Phi-3.5-mini-instruct_0shot_pilot.csv
│   ├── hasil_Qwen2.5-3B_0shot_pilot.csv
│   ├── hasil_SmolLM2-1.7B-Instruct_0shot_pilot.csv
│   ├── hasil_TinyLlama-1.1B-Chat-v1.0_0shot_pilot.csv
│   └── hasil_stablelm-zephyr-3b_0shot_pilot.csv
├── 01_data_audit.ipynb
├── 02_evaluasi_model.ipynb
├── 03_mitigasi_voting.ipynb
├── README.md
├── anotasi_78_soal_salah.xlsx
├── audit_59_soal_lanjutan.csv
├── audit_batch_tinyllama.csv
├── audit_taksonomi_tinyllama.csv
├── distribusi_kategori_kesalahan_tinyllama.csv
├── hasil_TinyLlama-1.1B-Chat-v1.0_0shot_pilot.csv
├── hasil_mitigasi_Phi-3.5-mini.csv
├── hasil_mitigasi_Qwen2.5-3B.csv
├── hasil_mitigasi_TinyLlama-1.1B.csv
├── laporan_audit.json
├── rekap_akurasi.csv
├── rekap_kategori_kesalahan.csv
├── rekap_mitigasi_semua_model.csv
├── rekap_per_kategori.csv
├── sains_bersih.csv
├── sains_bersih.xlsx
├── sampel_cek_kunci_manual.csv
├── sampel_cek_kunci_manual.xlsx
├── tabel_deskripsi.csv
└── tingkat_kesalahan_per_kategori_tinyllama.csv
```

## Cara Menjalankan

1. Buka *notebooks* di Google Colab (Runtime → GPU).
2. Jalankan `stem_err_id_preprocessing.ipynb` untuk membuat subset data `sains_bersih.csv`.
3. Buka `stem_err_id_evaluasi.ipynb`. Atur `NAMA_MODEL` di sel Konfigurasi. Untuk model yang butuh *login*, isi `HF_TOKEN` dengan token Hugging Face.
4. Jalankan sel dari atas ke bawah. Hasil tersimpan sebagai CSV.
5. Untuk menguji efektivitas mitigasi pada soal yang salah, gunakan `stem_err_id_mitigasi.ipynb`.

## Keterbatasan

- IndoMMLU sudah publik sejak 2023, sehingga model baru mungkin pernah melihatnya saat pelatihan.
- IndoMMLU tidak memiliki mapel Matematika. Cakupan proyek ini berfokus pada literasi **sains**.
- Eksperimen tahap mitigasi dan analisis kesalahan diuji pada sampel *pilot* (100 soal). Walau menunjukkan pola yang jelas, selisih akurasi antarkelompok dengan ukuran sangat kecil sebaiknya dilihat sebagai tren empiris.
- Data hanya berupa teks; soal yang aslinya merujuk gambar/tabel (sekitar ~1,4%) diuji tanpa menyertakan konteks visualnya.

## Tim (Sistem Informasi, Universitas Negeri Semarang)
- Annisa Yusri Nur Rochmah 
- Alya Fuji Insani 
- Fathimah Shaffa' Abqariyya 

```bibtex
@inproceedings{koto-etal-2023-indommlu,
  title     = {Large Language Models Only Pass Primary School Exams in {I}ndonesia: A Comprehensive Test on {I}ndo{MMLU}},
  author    = {Koto, Fajri and Aisyah, Nurul and Li, Haonan and Baldwin, Timothy},
  booktitle = {Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing},
  year      = {2023},
  url       = {[https://aclanthology.org/2023.emnlp-main.760/](https://aclanthology.org/2023.emnlp-main.760/)}
}
