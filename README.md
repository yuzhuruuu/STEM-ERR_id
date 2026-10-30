# STEM-Err-ID

**Evaluasi dan Mitigasi Kesalahan Large Language Model Skala Kecil pada Penyelesaian Soal Science Berbahasa Indonesia**

Karya untuk DIMAS-TI 2026, kategori Penambangan Data.

> **Status:** Selesai. Repositori ini memuat kode, hasil audit data, dan hasil eksperimen dari makalah final.

## Latar Belakang

Adopsi kecerdasan buatan dalam pendidikan bergerak lebih cepat daripada pemahaman tentang batas kepercayaannya, terutama untuk konteks kurikulum nasional. Proyek ini mengevaluasi kinerja model bahasa berskala kecil (*Small Language Models*/SLM dengan parameter ≤4B) dalam menyelesaikan soal sains berbahasa Indonesia. Kami memeriksa kinerja model secara empiris: seberapa akurat mereka, di soal mana mereka rentan salah, mengapa mereka salah (termasuk fenomena *overconfidence* dan kecenderungan memilih opsi tertentu), serta sejauh mana teknik mitigasi *Self-Consistency* (*majority voting*) mampu memperbaiki kesalahan tersebut.

## Tujuan Penelitian

1. Menyusun subset soal sains (IPA, Fisika, Kimia, dan Biologi) dari IndoMMLU yang diaudit dan dibersihkan sebagai sumber data yang dapat direplikasi.
2. Mengevaluasi akurasi lima SLM secara *zero-shot* dengan prosedur penilaian seragam.
3. Mengidentifikasi pola kesalahan prediksi model (muatan angka, panjang kalimat, kalibrasi *confidence*, dan kecenderungan pemilihan opsi).
4. Menguji efektivitas *Self-Consistency* (*majority voting*) dalam memperbaiki soal yang dijawab salah pada tahap *zero-shot*, serta membandingkannya antarmodel.

## Data

- **Subset sains IndoMMLU:** setelah audit dan pembersihan, diperoleh **2.925 soal sains bersih** (IPA, Fisika, Kimia, Biologi) dari jenjang SD hingga seleksi masuk PTN.
- **Sumber asli:** [indolem/IndoMMLU](https://huggingface.co/datasets/indolem/IndoMMLU) dan [fajri91/IndoMMLU](https://github.com/fajri91/IndoMMLU).
- **Lisensi:** CC BY-NC-SA 4.0 (atribusi wajib, non-komersial, pembagian ulang dengan lisensi yang sama).
- **Yang ada di repositori ini:** dataset mentah IndoMMLU **tidak** disertakan. Yang disertakan adalah subset turunan hasil pembersihan (`sains_bersih.csv` dan `sains_bersih.xlsx`), yang dibagikan dengan lisensi yang sama (CC BY-NC-SA 4.0) dan hanya untuk penggunaan non-komersial. Subset yang sama juga dapat dibuat ulang dari sumber asli dengan menjalankan `01_data_audit.ipynb`.

## Model

Model terbuka ≤4B parameter dari keluarga berbeda diuji secara *zero-shot*, dan tiga di antaranya dievaluasi lebih lanjut dengan *Self-Consistency*.

| Model | Parameter | Pendekatan Evaluasi |
|---|---|---|
| `Qwen2.5-3B` | ~3B | Zero-shot & Self-Consistency |
| `Phi-3.5-mini-instruct` | ~3,8B | Zero-shot & Self-Consistency |
| `StableLM-Zephyr-3B` | ~3B | Zero-shot |
| `SmolLM2-1.7B-Instruct` | ~1,7B | Zero-shot |
| `TinyLlama-1.1B-Chat-v1.0` | ~1,1B | Zero-shot & Self-Consistency |

## Metode

1. **Audit & preprocessing data:** pembersihan IndoMMLU dari soal duplikat, opsi kosong, kunci jawaban di luar rentang opsi, dan soal yang sama dengan kunci berbeda.
2. **Evaluasi awal (zero-shot):** menggunakan sampel uji coba (*pilot sample*) sebanyak 100 soal unik (acak sederhana, `random_state=42`). Model menilai probabilitas huruf jawaban (A–E).
3. **Analisis kesalahan:** tingkat kesalahan dikelompokkan menurut karakteristik soal (muatan angka vs konseptual, panjang kalimat), disertai analisis kalibrasi *confidence* dan kecenderungan pemilihan opsi.
4. **Mitigasi (Self-Consistency):** untuk soal yang salah pada tahap *zero-shot*, model membangkitkan 5 jawaban independen (*temperature* = 0,7), lalu *majority voting* menentukan jawaban akhir.

## Hasil

### 1. Performa Awal (Zero-Shot)

Pada pengujian 100 soal *pilot*:

* **Qwen2.5-3B**: 50,0%
* **Phi-3.5-mini**: 38,0% (*overconfidence*: skor keyakinan rata-rata 81,7% saat menjawab salah)
* **SmolLM2-1.7B**: 28,0% (memilih opsi C pada 52,0% soal)
* **StableLM-3B**: 26,0%
* **TinyLlama-1.1B**: 22,0% (setara tebakan acak, 21,9%)

*Catatan: pada empat dari lima model, tingkat kesalahan lebih tinggi pada soal bermuatan angka. Soal berkalimat panjang (>15 kata) juga lebih sering salah pada sebagian besar model, kecuali Qwen2.5-3B yang stabil.*

### 2. Efektivitas Mitigasi (Self-Consistency)

Mitigasi diuji hanya pada soal yang dijawab salah di tahap evaluasi awal:

| Model | Akurasi Awal | Akurasi Pasca-Mitigasi | Efektivitas Perbaikan |
|---|---|---|---|
| `Qwen2.5-3B` | 50,0% | 51,0% | **2,00%** (1 dari 50 soal salah diperbaiki) |
| `Phi-3.5-mini` | 38,0% | 46,0% | **12,90%** (8 dari 62 soal salah diperbaiki) |
| `TinyLlama-1.1B` | 22,0% | 27,0% | **6,41%** (5 dari 78 soal salah diperbaiki) |

**Kesimpulan utama:** *Self-Consistency* paling efektif meredam kecerobohan penalaran (*reasoning noise*) pada model kelas menengah seperti Phi-3.5-mini, tetapi hampir tidak mengoreksi ketidaktahuan konseptual (*knowledge error*) yang konsisten pada model terkuat seperti Qwen2.5-3B. Interpretasi jenis kesalahan ini masih berupa dugaan, karena jenis kesalahan belum dilabeli langsung pada ketiga model.

## Struktur Repositori

```text
├── arsip_pilot/                                   # hasil zero-shot mentah tiap model (sampel pilot)
│   ├── hasil_Phi-3.5-mini-instruct_0shot_pilot.csv
│   ├── hasil_Qwen2.5-3B_0shot_pilot.csv
│   ├── hasil_SmolLM2-1.7B-Instruct_0shot_pilot.csv
│   ├── hasil_TinyLlama-1.1B-Chat-v1.0_0shot_pilot.csv
│   └── hasil_stablelm-zephyr-3b_0shot_pilot.csv
├── 01_data_audit.ipynb                            # audit & pembersihan data
├── 02_evaluasi_model.ipynb                        # evaluasi zero-shot
├── 03_mitigasi_voting.ipynb                       # mitigasi Self-Consistency
├── README.md
├── sains_bersih.csv / sains_bersih.xlsx           # subset 2.925 soal sains bersih
├── laporan_audit.json                             # ringkasan hasil audit
├── tabel_deskripsi.csv                            # komposisi per mata pelajaran & jenjang
├── sampel_cek_kunci_manual.csv / .xlsx            # sampel 30 soal untuk cek kunci manual
├── hasil_TinyLlama-1.1B-Chat-v1.0_0shot_pilot.csv
├── rekap_akurasi.csv                              # rekap akurasi semua model
├── rekap_per_kategori.csv                         # akurasi per kategori
├── hasil_mitigasi_Qwen2.5-3B.csv
├── hasil_mitigasi_Phi-3.5-mini.csv
├── hasil_mitigasi_TinyLlama-1.1B.csv
├── rekap_mitigasi_semua_model.csv                 # rekap efektivitas mitigasi
├── anotasi_78_soal_salah.xlsx                     # anotasi manual soal salah TinyLlama
├── audit_batch_tinyllama.csv
├── audit_59_soal_lanjutan.csv
├── audit_taksonomi_tinyllama.csv
├── rekap_kategori_kesalahan.csv
├── distribusi_kategori_kesalahan_tinyllama.csv
└── tingkat_kesalahan_per_kategori_tinyllama.csv
```

## Cara Menjalankan

1. Buka *notebook* di Google Colab (Runtime → Change runtime type → GPU).
2. Jalankan `01_data_audit.ipynb` untuk mengunduh IndoMMLU dan membuat subset `sains_bersih.csv` (2.925 soal) beserta `laporan_audit.json`.
3. Buka `02_evaluasi_model.ipynb`. Atur `NAMA_MODEL` pada sel Konfigurasi. Untuk model yang membutuhkan *login*, isi `HF_TOKEN` dengan token Hugging Face milik sendiri.
4. Jalankan sel dari atas ke bawah. Hasil tersimpan sebagai CSV. Ulangi untuk setiap model.
5. Untuk menguji efektivitas mitigasi pada soal yang salah, jalankan `03_mitigasi_voting.ipynb`.

## Keterbatasan

- IndoMMLU sudah publik sejak 2023, sehingga model baru mungkin pernah melihat soalnya saat pelatihan.
- IndoMMLU tidak memiliki mata pelajaran Matematika, sehingga cakupan proyek ini terbatas pada **sains**.
- Analisis kesalahan dan mitigasi diuji pada sampel *pilot* (100 soal). Hasilnya dibaca sebagai studi pendahuluan dan tren empiris, bukan kesimpulan yang dapat digeneralisasikan ke seluruh 2.925 soal.
- Pembersihan data hanya memeriksa format dan konsistensi internal. Pengecekan kebenaran kunci dilakukan manual pada 30 soal saja (28 sesuai).
- Mitigasi hanya diuji pada soal yang salah, sehingga kemungkinan *voting* mengubah jawaban benar menjadi salah belum terukur.
- Data hanya berupa teks. Soal yang merujuk gambar atau tabel (tidak lebih dari ~1,4% pada tiap kelompok) diuji tanpa konteks visualnya.

## Tim (Sistem Informasi, Universitas Negeri Semarang)

- Annisa Yusri Nur Rochmah
- Alya Fuji Insani
- Fathimah Shaffa' Abqariyya

## Sitasi

Jika menggunakan data IndoMMLU, mohon sitasi karya aslinya:

```bibtex
@inproceedings{koto-etal-2023-indommlu,
  title     = {Large Language Models Only Pass Primary School Exams in {I}ndonesia: A Comprehensive Test on {I}ndo{MMLU}},
  author    = {Koto, Fajri and Aisyah, Nurul and Li, Haonan and Baldwin, Timothy},
  booktitle = {Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing},
  year      = {2023},
  url       = {https://aclanthology.org/2023.emnlp-main.760/}
}
```
