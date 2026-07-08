# Customer Feedback Sentiment Analyzer Agent

Capstone Project — IBM x Hackative8 Data AI Prompting Bootcamp

## Problem Statement

Tim produk dan bisnis kesulitan memantau ulasan pelanggan karena volume data yang besar, sehingga membutuhkan waktu lama untuk memahami insight-nya. Jika dibiarkan, keterlambatan penanganan ini berisiko menurunkan reputasi brand.

## Tujuan

Membangun AI Agent yang membantu tim produk dan bisnis memahami ulasan pelanggan secara cepat melalui:
- Ringkasan naratif yang kohesif
- Klasifikasi sentimen (Positive, Neutral, Negative)
- Ringkasan risiko bisnis tingkat eksekutif
- Rekomendasi perbaikan yang actionable

Ditampilkan dalam bentuk dashboard interaktif.

## Target Pengguna

Tim produk dan tim bisnis.

## Fungsi Utama AI Agent

1. Automated Survey Sentiment Analysis
2. Customer Service Complaint Classification
3. Executive-Level Business Risk Summary
4. Actionable Improvement Recommendation

## Arsitektur: 3-Layer Prompting

| Layer | Kapan Berjalan | Output | Yang Menghitung |
|---|---|---|---|
| **Layer 1 — Sentiment & Theme Extraction** | Per baris (1.001×) | `sentiment`, `primary_theme` | Mock Granite (LLM) |
| **Layer 2 — Risk & Recommendation Extraction** | Per baris, hanya NEGATIVE/NEUTRAL (152×) | `risk_summary`, `recommendations` | Mock Granite (LLM) |
| **Layer 3 — Executive Aggregation** | 1× setelah seluruh baris selesai | `narrative_summary`, `top_recommendations`, `overall_risk_level`, `top5_themes` | Python murni → angka dikirim ke LLM sebagai string literal |

**Prinsip utama:** semua perhitungan numerik (counting, ranking, threshold risk level) dilakukan Python murni — LLM hanya bertugas menyusun narasi/klasifikasi dari data yang sudah pasti, untuk mencegah hallucination pada angka.

## Tech Stack

- Python 3.x
- Pandas (pemrosesan data CSV)
- Streamlit (dashboard interaktif)
- Mock simulation engine (menggantikan koneksi live IBM watsonx/Granite yang tidak tersedia di environment)
- IBM Bob (AI development partner untuk generate `app.py`)

## Struktur Folder

```
├── mock_analyzer.py        # Script utama: Layer 1 + Layer 2 + Layer 3
├── app.py                  # Dashboard Streamlit
├── laptop_online_review.csv  # Dataset ulasan pelanggan
├── analysis_result.json    # Output hasil analisis (data source dashboard)
├── requirements.txt        # Daftar dependency
├── docs/
│   └── prompt_journal.md   # Dokumentasi evolusi prompt engineering
└── README.md
```

## Cara Menjalankan

1. Clone repository ini:
   ```bash
   git clone https://github.com/USERNAME/NAMA-REPO.git
   cd NAMA-REPO
   ```

2. Install dependency:
   ```bash
   pip install -r requirements.txt
   ```

3. Jalankan analisis (generate `analysis_result.json`):
   ```bash
   python mock_analyzer.py
   ```

4. Jalankan dashboard:
   ```bash
   streamlit run app.py
   ```
   Jika muncul error "streamlit not recognized", gunakan:
   ```bash
   python -m streamlit run app.py
   ```

5. Dashboard akan terbuka otomatis di browser pada `http://localhost:8501`

## Catatan Prototype

Project ini dikembangkan dalam **mode simulasi (mock/offline)** karena akun IBM Cloud/watsonx tidak terkoneksi live pada environment pengembangan. Seluruh logika arsitektur (Layer 1, 2, 3) tetap merepresentasikan struktur prompting penuh dan siap diintegrasikan dengan model live (`ibm/granite-13b-instruct-v2`) apabila koneksi API tersedia.

## Dokumentasi Prompt Engineering

Proses pengembangan prompt dilakukan secara iteratif, dari versi single-layer naive hingga arsitektur 3-layer final. Detail lengkap perubahan, alasan, dan dampaknya terhadap kualitas output ada di [`docs/prompt_journal.md`](docs/prompt_journal.md).

## Author

Elsa Fitriani — IBM x Hackative8 Data AI Prompting Bootcamp

