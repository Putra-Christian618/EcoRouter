# EcoRouter AI: Payload-Aware Logistics

**Kategori Inovasi:** Smart Logistics  

EcoRouter AI adalah program (*Minimum Viable Product*) sistem manajemen kurir yang dirancang untuk mengatasi inefisiensi konsumsi bahan bakar pada rute jarak pendek dengan muatan berat. Berbeda dengan sistem perutean konvensional yang hanya mencari jarak terpendek (*shortest path*), EcoRouter menggunakan fungsi objektif **Ton-Kilometer**. Sistem ini memastikan kendaraan berat tidak menahan muatan maksimal dalam waktu lama, sekaligus mengotomatisasi pendataan gudang melalui *Computer Vision* dan mengeliminasi proses *re-handling* kurir dengan instruksi muat LIFO (*Last In, First Out*).

---

## Fitur Utama (MVP Scope)
Sistem ini berfokus pada alur interaksi inti yang sinkron (Input Pengguna $\rightarrow$ Output AI) tanpa *background jobs* atau arsitektur *database* terdistribusi yang kompleks:
1. **Visual Ingestion (Input AI):** Pengguna memindai kargo melalui kamera langsung (*webcam*) atau unggah fail gambar. Model *Computer Vision* secara otomatis menghitung jumlah paket.
2. **Ton-Kilometer Routing Engine (Proses AI):** Berdasarkan jumlah paket, sistem menarik metadata fisik (kg & cm³) dari pangkalan data dan mengkalkulasi rute paling hemat bahan bakar menggunakan algoritma *Capacitated Vehicle Routing Problem* (CVRP).
3. **Frictionless Unloading (Output):** Mencetak urutan muat barang (*loading sequence*) secara otomatis menggunakan prinsip LIFO.
4. **Interactive Telemetry:** Dasbor *real-time* yang menampilkan persentase efisiensi bahan bakar, konversi penghematan biaya (IDR), dan visualisasi peta rute komparatif.

---

## Tech Stack
*   **Frontend / Antarmuka:** Streamlit, Folium (Interactive Mapping)
*   **Backend / Integrasi:** Python 3.12, Requests (OSRM API Integration)
*   **Model AI & Algoritma:** 
    *   Ultralytics YOLOv11n (*Headless*) untuk *Object Detection*.
    *   Google OR-Tools untuk optimasi matematis jarak dan muatan.
    *   Haversine Formula (sebagai *offline fail-safe/fallback*).

---

## Arsitektur Sistem Global
EcoRouter menggunakan pendekatan modular yang dieksekusi secara sinkron dalam satu alur *pipeline* *end-to-end*:
1. **Fase Visi Komputer:** Gambar kargo dikirim ke model YOLOv11n yang dimuat di dalam memori untuk mendeteksi *bounding box*. Jumlah kotak dihitung dan diteruskan ke modul *data wrangling*.
2. **Fase Ingesti Basis Data (*Cache Memory*):** Sistem tidak membebani *server* dengan pemanggilan *database* eksternal. Angka hitungan YOLO memicu ekstraksi $N$ data metadata logistik (berat, volume, destinasi Jabodetabek) dari modifikasi *Olist E-Commerce Dataset* yang telah disimpan ke dalam *in-memory cache*.
3. **Fase Optimasi Rute:** Koordinat dikirim ke OSRM API untuk mendapatkan matriks jarak nyata. Matriks ini dimasukkan ke *Cost Evaluator* Google OR-Tools yang telah dimodifikasi dengan penalti beban linier. Hasil rute optimum dikembalikan ke *Frontend* Streamlit untuk divisualisasikan.

---

## Prerequisites
Untuk menjalankan aplikasi ini secara lokal di mesin Anda, pastikan Anda telah memasang:
*   **Docker** (versi 20.10.0 atau lebih baru)
*   **Docker Compose** (versi v2 atau lebih baru)
*   *Catatan: Sistem ini menggunakan OSRM API publik dan algoritma Haversine luring, sehingga **TIDAK** membutuhkan konfigurasi fail `.env` atau API Key rahasia apa pun.*

---

## Setup Guide / Cara Menjalankan Aplikasi
Ikuti langkah-langkah minimalis berikut untuk menjalankan purwarupa secara lokal menggunakan *Docker Compose*:

1. **Kloning Repositori:**
   ```bash
   git clone git clone https://github.com/Putra-Christian618/EcoRouter.git
   cd Project1
2. **Jalankan Kontainer:**
   Jalankan perintah berikut untuk mengunduh semua dependencies (Streamlit, PyTorch, OR-Tools) dan menyalakan server:
   ```bash
   docker compose up --build
3. **Akses Aplikasi:**
   Tunggu hingga terminal menampilkan status running. Buka peramban (web browser) Anda dan akses tautan berikut: [Localhost Aplikasi](http://localhost:8501/)

## Dataset & Model
1. Dataset Deteksi Paket: Model dilatih (fine-tuning) secara lokal menggunakan "Warehouse Delivery Box Detection Dataset" publik, dengan menerapkan augmentasi data agresif (Mosaic, Mixup, HSV) untuk mencegah overfitting pada variasi pencahayaan.
2. Dataset Logistik: Katalog profil berat, volume, dan koordinat disimulasikan menggunakan modifikasi dan pembersihan ketat dari Olist Brazilian E-Commerce Dataset.
3. Model Al: Pre-trained YOLOv11n, dilatih ulang (fine-tuned) selama 100 epochs memanfaatkan akselerasi CUDA pada lingkungan WSL lokal.
