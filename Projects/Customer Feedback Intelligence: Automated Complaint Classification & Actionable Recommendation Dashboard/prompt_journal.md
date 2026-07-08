# Prompt Engineering Journal

Dokumentasi evolusi prompt selama pengembangan Customer Feedback Sentiment Analyzer Agent, dari draft awal hingga arsitektur final 3-layer.

---

## Iterasi 1 — Single-Layer Prompt (Draft Awal)

**Prompt yang digunakan:** meminta LLM membaca CSV, lalu langsung menghasilkan sentiment, tema, risk summary, dan recommendation untuk setiap baris dalam satu panggilan.

**Masalah yang ditemukan:**
1. **Hallucination pada agregasi matematika** — LLM diminta menghitung total review, hasilnya salah (1002, padahal data asli 1001 baris). LLM adalah generative text model, bukan mesin hitung.
2. **Risk & recommendation bocor ke ulasan positif** — LLM tetap mengeluarkan `risk_summary` dan `recommendations` untuk ulasan positif ("Best under 60k..."), padahal ulasan positif tidak memerlukan manajemen risiko. Ini boros token dan tidak efisien.

---

## Iterasi 2 — Layered Prompting + Mock Engine

**Perubahan:**
- Seluruh perhitungan counter (POSITIVE/NEGATIVE/NEUTRAL) dipindah ke Python murni (`.iterrows()`), bukan diserahkan ke LLM.
- Ditambahkan logic kondisional: Layer 2 hanya dijalankan jika Layer 1 = NEGATIVE/NEUTRAL; jika POSITIVE, otomatis diisi "No risk identified" tanpa panggilan LLM tambahan.
- Dibangun `mock_llm_response()` untuk simulasi output model, karena akun tidak terkoneksi live ke IBM Cloud/watsonx.

**Alasan:** Mengatasi hallucination angka dan pemborosan token dari Iterasi 1, sekaligus membuat prototype tetap bisa didemokan tanpa dependency ke koneksi cloud live.

**Dampak:** Akurasi angka menjadi 100% (tidak ada lagi selisih hitung). Penggunaan token lebih efisien karena Layer 2 tidak dipanggil untuk data yang tidak relevan. Prototype menjadi reproducible di environment tanpa koneksi live.

---

## Iterasi 3 — Penambahan Executive Aggregation Layer (Layer 3)

**Masalah yang ditemukan:** Arsitektur 2-layer hanya menghasilkan insight per baris ulasan. Ini belum menjawab dua dari empat fungsi utama Agent yang ditargetkan sejak awal: *Executive-Level Business Risk Summary* dan *Actionable Improvement Recommendation* di tingkat keseluruhan (bukan per ulasan individual).

**Perubahan:**
- Ditambahkan Layer 3 yang berjalan satu kali di akhir loop, mengagregasi seluruh hasil Layer 1 & 2.
- Top-5 tema komplain (`top5_themes`) dihitung menggunakan `collections.Counter`, dan `overall_risk_level` ditentukan melalui rule threshold Python (>20% negative = HIGH, 10–20% = MEDIUM, <10% = LOW) — bukan diserahkan ke LLM untuk dihitung atau diranking.
- Ditambahkan guardrail eksplisit: jika narasi Layer 3 menyebut angka spesifik, angka tersebut wajib disisipkan dari variabel Python via string template (`.format()`), bukan dikarang bebas oleh LLM.

**Alasan:** Mencegah pengulangan masalah hallucination yang sama seperti Iterasi 1, sekaligus melengkapi cakupan fungsi Agent sesuai problem statement.

**Dampak:** Agent menghasilkan insight di dua level sekaligus — detail per ulasan (Layer 1+2) dan ringkasan strategis siap pakai untuk pengambilan keputusan (Layer 3) — dengan akurasi numerik yang tetap terjaga karena prinsip "Python menghitung, LLM menarasikan" diterapkan konsisten di semua layer.

---

## Iterasi 4 — Pemisahan Logic Layer dan Presentation Layer

**Masalah yang berpotensi terjadi:** Jika perhitungan persentase/risk level dilakukan ulang di dashboard (`app.py`), ada risiko duplikasi logic dan inkonsistensi angka antara file analisis dan file dashboard.

**Perubahan:** `app.py` didesain untuk murni membaca dan menampilkan data dari `analysis_result.json`, tanpa kalkulasi ulang apa pun. Seluruh angka final (persentase, risk level, top themes) difinalisasi di `mock_analyzer.py` sebagai satu sumber kebenaran (single source of truth).

**Dampak:** Konsistensi data terjamin antara backend logic dan dashboard visual. Pemisahan concern yang jelas antara "layer yang menghitung" dan "layer yang menampilkan" memudahkan debugging dan pengembangan lebih lanjut.

---

## Ringkasan Prinsip yang Dipegang Konsisten

1. **Python menghitung, LLM menarasikan** — semua operasi matematis (counting, ranking, threshold) tidak pernah diserahkan ke LLM.
2. **Efisiensi token melalui conditional layer** — layer lanjutan hanya dipanggil jika benar-benar relevan (misalnya Layer 2 hanya untuk NEGATIVE/NEUTRAL).
3. **Single source of truth** — hasil akhir difinalisasi di satu tempat (`analysis_result.json`), dashboard hanya menampilkan.
4. **Guardrail terhadap angka dalam teks naratif** — angka yang muncul di narasi LLM selalu disuntik dari variabel yang sudah diverifikasi, bukan digenerate bebas.
