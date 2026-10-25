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
