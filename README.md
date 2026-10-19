# STEM-Err-ID

**Membedah kapan dan mengapa model bahasa kecil salah menjawab soal sains berbahasa Indonesia, serta cara menguranginya.**

Karya untuk DIMAS-TI 2026, kategori Penambangan Data.

> **Status:** dalam pengerjaan. Bagian bertanda `TBD` akan diisi setelah eksperimen selesai.

## Latar belakang

Sekolah mulai memakai AI dalam pembelajaran, tetapi belum ada peta yang jelas tentang kapan jawaban AI untuk soal sains berbahasa Indonesia layak dipercaya. Proyek ini tidak membuat model baru. Kami **memeriksa** model terbuka berukuran kecil: di soal mana mereka salah, mengapa salah, dan apakah kesalahan itu bisa diprediksi dan dikurangi.

## Pertanyaan penelitian

1. Seberapa akurat model terbuka kecil (≤4B parameter) pada soal sains berbahasa Indonesia, per mapel dan jenjang?
2. Apa jenis kesalahan yang paling sering terjadi (misalnya hitungan multi-langkah, membaca tabel/grafik dalam teks, konversi satuan, istilah lokal, konsep vs hafalan)?
3. Bisakah soal yang rawan salah diprediksi dari ciri soalnya, sebagai "lampu kuning" untuk guru?
4. Seberapa besar perbaikan dari mitigasi ringan (self-consistency, jalur kalkulator/Python), dan pada kategori kesalahan yang mana?

## Data

- **IndoMMLU**: soal pilihan ganda dari SD sampai seleksi PTN. Kami hanya memakai subset sains (IPA, Fisika, Kimia, Biologi).
- Sumber: [indolem/IndoMMLU](https://huggingface.co/datasets/indolem/IndoMMLU) dan [fajri91/IndoMMLU](https://github.com/fajri91/IndoMMLU).
- Lisensi: kartu data menyebut CC BY-NC-SA 4.0 (metadata Hugging Face menulis MIT). Kami mengikuti yang lebih ketat: atribusi wajib dan non-komersial.
- Data **tidak** disertakan di repo ini. Unduh dari sumber aslinya. Tanggal unduh: `TBD`.

## Model

Model terbuka ≤4B parameter dari keluarga berbeda, dengan prompt seragam, diuji 0-shot dan few-shot.

| Model | Keluarga | Versi/tanggal unduh |
|---|---|---|
| `TBD` | `TBD` | `TBD` |
| `TBD` | `TBD` | `TBD` |
| `TBD` | `TBD` | `TBD` |
| `TBD` | `TBD` | `TBD` |

## Metode

1. **Evaluasi:** model menilai peluang huruf jawaban (A–E) untuk tiap soal, lalu dibandingkan otomatis dengan kunci jawaban. Peluang tiap opsi disimpan untuk analisis kalibrasi.
2. **Akurasi:** dirinci per mapel dan jenjang, dibandingkan dengan leaderboard IndoMMLU.
3. **Error mining:** soal yang salah di ≥2 model diubah menjadi embedding, diklasterkan, lalu 100–150 soal dilabeli manual menjadi taksonomi kesalahan. Kesepakatan antar-pelabel dihitung dengan Cohen's kappa.
4. **Prediktor kerawanan:** model ringan (logistic regression atau gradient boosting) memprediksi peluang model salah dari ciri soal.
5. **Mitigasi:** self-consistency dan/atau jalur kalkulator/Python untuk soal hitungan. Gain diukur per kategori kesalahan.
6. **Uji kontaminasi:** urutan opsi jawaban diacak lalu akurasi dihitung ulang, untuk melihat tanda model menghafal data uji.

**Metrik:** akurasi per jenjang/mapel, macro-F1 dan AUC prediktor, ECE/Brier score (kalibrasi), selisih akurasi mitigasi dengan CI bootstrap dan uji McNemar.

## Hasil

`TBD`

| Model | 0-shot | Few-shot |
|---|---|---|
| `TBD` | `TBD` | `TBD` |

## Struktur repo

```
.
├── notebooks/
│   └── stem_err_id_evaluasi.ipynb   # evaluasi model pada soal sains IndoMMLU
├── results/                          # CSV hasil per model (id soal, jawaban, benar/salah, peluang)
├── labels/                           # label manual taksonomi kesalahan
└── README.md
```

## Cara menjalankan

1. Buka `notebooks/stem_err_id_evaluasi.ipynb` di Google Colab (Runtime → GPU).
2. Atur `NAMA_MODEL`, `N_SHOT`, dan `PILOT` di sel Konfigurasi.
3. Jalankan sel dari atas ke bawah. Hasil tersimpan sebagai CSV di Google Drive. Untuk model yang butuh login (misalnya Gemma), isi `HF_TOKEN` dengan token Hugging Face.
4. Cek nama kolom di `COL` terhadap `df.columns` sebelum menjalankan penuh.

Mulai dengan `PILOT = 100`, lalu ganti ke `None` untuk semua soal sains.

## Keterbatasan

- IndoMMLU sudah publik sejak 2023, sehingga model baru mungkin pernah melihatnya saat pelatihan. Hasil uji kontaminasi dilaporkan di bagian hasil.
- IndoMMLU tidak memiliki mapel Matematika. Cakupan proyek ini adalah **sains**, bukan matematika.
- Jumlah soal per mapel dan jenjang tidak merata, sehingga klaim per kategori hanya dibuat jika sampelnya cukup.
- Kami tidak mengklaim hasil terbaik (state-of-the-art), hanya perbandingan pada kondisi yang seragam.

## Sitasi

Jika memakai IndoMMLU, kutip paper aslinya:

```bibtex
@inproceedings{koto-etal-2023-indommlu,
  title     = {Large Language Models Only Pass Primary School Exams in {I}ndonesia: A Comprehensive Test on {I}ndo{MMLU}},
  author    = {Koto, Fajri and Aisyah, Nurul and Li, Haonan and Baldwin, Timothy},
  booktitle = {Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing},
  year      = {2023},
  url       = {https://aclanthology.org/2023.emnlp-main.760/}
}
```

## Tim

- `Yusri`
- `Shaffa`
- `Fuji`

## Lisensi

TIM tiba-tiba dimasti
