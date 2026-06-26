# INSTALASI.md

## 1. Prasyarat

| Tools | Versi yang disarankan | Cek instalasi |
|---|---|---|
| Python | 3.9 – 3.11 | `python --version` |
| pip | terbaru | `pip --version` |

> **Catatan:** `lightgbm` dan `xgboost` butuh compiler C++ di beberapa OS. Jika instalasi gagal di Windows, install dulu [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/). Di Linux biasanya langsung berhasil lewat `pip` (wheel sudah tersedia).

---

## 2. Siapkan Folder Project

```
stunting-detector/
├── ml_develop.ipynb
├── app.py
├── design.py
├── raw_dataset.csv         # dataset mentah (dibutuhkan HANYA jika ingin training ulang)
├── model_artifacts/        # akan terisi otomatis setelah notebook dijalankan
└── requirements.txt
```

```bash
mkdir stunting-detector
cd stunting-detector
# letakkan ml_develop.ipynb, app.py, design.py di folder ini
```

---

## 3. Setup Virtual Environment & Dependencies

```bash
python -m venv venv

# Aktifkan:
source venv/bin/activate        # macOS/Linux
venv\Scripts\activate           # Windows
```

Buat file `requirements.txt`:

```txt
pandas
numpy
scikit-learn
matplotlib
seaborn
xgboost
lightgbm
imbalanced-learn
optuna
shap
joblib
fastapi
uvicorn
pydantic
streamlit
jupyter
```

Install:
```bash
pip install -r requirements.txt
```

---

## 4. Mendapatkan Model — Pilih Salah Satu

Anda butuh folder **`model_artifacts/`** berisi 4 file (`stunting_model.pkl`, `scaler.pkl`, `label_encoder.pkl`, `metadata.json`) sebelum `app.py` atau `design.py` bisa berjalan. Ada 2 cara:

### Opsi A — Sudah punya `model_artifacts/` dari sebelumnya
Cukup letakkan folder tersebut di root project (sejajar dengan `app.py`), lalu lanjut ke **Langkah 5**.

### Opsi B — Latih ulang dari awal via `ml_develop.ipynb`
1. Siapkan dataset mentah dengan nama **`raw_dataset.csv`** (format CSV separator `;`) di root project — sesuai yang dibaca notebook:
   ```python
   DATA_PATH = 'raw_dataset.csv'
   df_raw = pd.read_csv(DATA_PATH, sep=';')
   ```
2. Jalankan Jupyter:
   ```bash
   jupyter notebook ml_develop.ipynb
   ```
3. Jalankan seluruh cell secara berurutan (**Run All**). Notebook akan:
   - Membersihkan data & melakukan feature engineering (BMI, velocity berat/tinggi, encoding, dll).
   - Melakukan seleksi fitur dengan **Mutual Information**.
   - Menyeimbangkan kelas dengan **SMOTE**.
   - Melakukan tuning hyperparameter dengan **Optuna** dan membandingkan beberapa model (Logistic Regression, Decision Tree, Random Forest, Extra Trees, Gradient Boosting, XGBoost, **LightGBM**).
   - Menyimpan model terbaik (LightGBM) beserta scaler, label encoder, dan metadata ke folder **`model_artifacts/`**.
4. Setelah selesai, akan muncul ringkasan akhir berisi metrik model dan konfirmasi artefak tersimpan.

> Proses tuning Optuna + SHAP bisa memakan waktu beberapa menit tergantung spesifikasi komputer.

---

## 5. Verifikasi Artefak Model

Pastikan folder berikut sudah ada dan terisi:
```
model_artifacts/
├── stunting_model.pkl
├── scaler.pkl
├── label_encoder.pkl
└── metadata.json
```

Cek cepat lewat terminal:
```bash
ls model_artifacts/
```

---

## 6. Menjalankan API (`app.py`)

```bash
python app.py
```
atau dengan auto-reload saat development:
```bash
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

Jika berhasil, akan tampil log:
```
✅ Model LightGBM berhasil dimuat
   Fitur model : 12 dari 14 total
   Kelas (4): ['Normal', 'Risiko Stunting Ringan', 'Risiko Stunting Sedang', 'Risiko Stunting Tinggi']
```

API berjalan di `http://localhost:8000`. Dokumentasi interaktif (Swagger UI) otomatis tersedia di:
```
http://localhost:8000/docs
```

### Contoh test endpoint `/predict` via `curl`:
```bash
curl -X POST http://localhost:8000/predict \
  -H "Content-Type: application/json" \
  -d '{
        "bb_lahir": 3.0,
        "tb_lahir": 49.0,
        "bb": 9.0,
        "tb": 77.0,
        "lila": 13.5,
        "umur": 18,
        "jenis_kelamin": "Laki-laki"
      }'
```

### Cek kesehatan service:
```bash
curl http://localhost:8000/health
```

---

## 7. Menjalankan Dashboard (`design.py`)

Dashboard ini **berdiri sendiri** (tidak memanggil API di Langkah 6 — ia memuat model langsung), jadi cukup jalankan:

```bash
streamlit run design.py
```

Browser otomatis terbuka ke `http://localhost:8501`, dengan 2 halaman:
- **🔍 Pendeteksi Stunting** — form input data antropometri balita → hasil klasifikasi & probabilitas.
- **📊 Hasil Pengembangan Model** — metrik evaluasi model & daftar fitur terpilih, dibaca dari `model_artifacts/metadata.json`.

> Anda bisa menjalankan API (Langkah 6) dan dashboard (Langkah 7) secara bersamaan di terminal yang berbeda — keduanya independen dan tidak saling bergantung.

---

## 8. Ringkasan Urutan Menjalankan dari Nol

```bash
# 1. Setup environment
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# 2. Siapkan model_artifacts/ — pilih salah satu:
#    a) salin folder model_artifacts yang sudah ada, ATAU
#    b) jalankan notebook training:
jupyter notebook ml_develop.ipynb     # Run All (butuh raw_dataset.csv)

# 3. Jalankan API
uvicorn app:app --reload --port 8000

# 4. (Terminal baru) Jalankan dashboard
streamlit run design.py
```
