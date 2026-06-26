# INSTALASI.md

Panduan menjalankan **Sistem Deteksi Dini Risiko Stunting** di komputer lokal. Project ini terdiri dari 3 bagian:

1. `ml_develop.ipynb` - notebook untuk melatih model (LightGBM, dengan SMOTE & seleksi fitur) dan menyimpan artefak ke folder `model_artifacts/`.
2. `app.py` - REST API berbasis FastAPI yang memuat artefak model dan menyediakan endpoint untuk mendeteksi risiko stunting.
3. `design.py` - dashboard Streamlit yang juga memuat artefak model secara langsung (tanpa lewat API) untuk antarmuka input form + visualisasi hasil training.

> Tidak ada database yang dipakai di project ini. Ketiga file hanya bergantung pada dataset CSV lokal (untuk training) dan artefak model (`.pkl` / `.json`) hasil training - tidak ada koneksi PostgreSQL/MongoDB/dsb seperti pada project lain.
>
> `app.py` menyebut "menerima data dari backend Go"  itu hanya konteks bahwa di production API ini dipanggil oleh service Go lain. Untuk menjalankan di lokal, Anda tidak perlu menyiapkan backend Go tersebut; cukup jalankan `app.py` dan panggil endpointnya langsung.


## 1. Prasyarat


1. Python versi 3.9 - 3.11 
cara cek `python --version`

2. pip versi terbaru 
cara cek `pip --version`

## 2. Siapkan Folder Project
mkdir stunting-detector

# letakkan ml_develop.ipynb, app.py, design.py di folder ini


## 3. Setup Virtual Environment & Dependencies

python -m venv venv

Jalankan `requirements.txt`:

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

command: pip install -r requirements.txt


## 6. Menjalankan API (`app.py`)
python app.py

atau dengan auto-reload saat development:
uvicorn app:app --reload --host 0.0.0.0 --port 8000


## 7. Menjalankan Dashboard (`design.py`)

Dashboard ini berdiri sendiri, jadi cukup jalankan:
streamlit run design.py


Browser otomatis terbuka ke `http://localhost:8501`, dengan 2 halaman:
- Pendeteksi Stunting - form input data antropometri balita ->  hasil klasifikasi & probabilitas.
- Hasil Pengembangan Model - metrik evaluasi model & daftar fitur terpilih, dibaca dari `model_artifacts/metadata.json`.

> Anda bisa menjalankan API (Langkah 6) dan dashboard (Langkah 7) secara bersamaan di terminal yang berbeda — keduanya independen dan tidak saling bergantung.

## 9. Ringkasan Urutan Menjalankan dari Nol

# 1. Setup environment
pip install -r requirements.txt

# 2. Siapkan model_artifacts/ — pilih salah satu:
#    jalankan notebook training:

# 3. Jalankan API
uvicorn app:app --reload --port 8000

# 4. (Terminal baru) Jalankan dashboard
streamlit run design.py