# EcoRouter AI

**Inovation Category:** Smart Logistics  

EcoRouter AI is a courier management system (Minimum Viable Product) designed to address fuel consumption inefficiencies on short-haul routes involving heavy loads. Unlike conventional routing systems that merely seek the shortest distance, EcoRouter employs a **Ton-Kilometer** objective function. The system ensures heavy vehicles do not carry maximum loads for extended periods, while also automating warehouse data capture via Computer Vision and eliminating courier re-handling through LIFO (Last In, First Out) loading instructions.
---

## Main Feature (MVP Scope)
This system focuses on the core synchronous interaction flow (User Input → AI Output) without background jobs or complex distributed database architectures:
1. **Visual Ingestion (AI input):** Users scan the cargo using a live camera (webcam) or by uploading an image file. A computer vision model automatically counts the number of packages.
2. **Ton-Kilometer Routing Engine (AI Process):** Based on the number of packages, the system retrieves physical metadata (kg & cm³) from the database and calculates the most fuel-efficient route using the *Capacitated Vehicle Routing Problem* (CVRP) algorithm.
3. **Frictionless Unloading (Output):** Automatically generate the cargo loading sequence using the LIFO principle.
4. **Interactive Telemetry:** A real-time dashboard displaying fuel efficiency percentage, cost savings conversion (IDR), and comparative route map visualizations.

---

## Tech Stack
*   **Frontend / Antarmuka:** Streamlit, Folium (Interactive Mapping)
*   **Backend / Integrasi:** Python 3.12, Requests (OSRM API Integration)
*   **Model AI & Algoritma:** 
    *   Ultralytics YOLOv11n (*Headless*) for *Object Detection*.
    *   Google OR-Tools for the mathematical optimization of distance and load.
    *   Haversine Formula (as an offline fail-safe/fallback).

---

## Global System Architecture
EcoRouter employs a modular approach executed synchronously within a single end-to-end pipeline:
1. **Computer Vision Phase:** Cargo images are sent to the YOLOv11n model loaded in memory to detect bounding boxes. The number of boxes is calculated and passed to the data wrangling module.
2. **Database Ingestion Phase (Cache Memory):** The system avoids burdening the server with external database calls. The YOLO detection count triggers the extraction of $N$ logistical metadata records (weight, volume, Jabodetabek destination) from a modified version of the *Olist E-Commerce Dataset* stored in the in-memory cache.
3. **Route Optimization Phase:** Coordinates are sent to the OSRM API to obtain a real-world distance matrix. This matrix is ​​fed into the Google OR-Tools *Cost Evaluator*, which has been modified to include a linear load penalty. The resulting optimal route is returned to the Streamlit *frontend* for visualization.

---

## Prerequisites
To run this application locally on your machine, ensure you have installed:
*   **Docker** (version 20.10.0 or later)
*   **Docker Compose** (version v2 or later)
*   *Note: This system uses the public OSRM API and an offline Haversine algorithm, so it does **NOT** require a `.env` file configuration or any secret API keys.*

---

## Setup Guide
Follow these minimalist steps to run the prototype locally using Docker Compose:

1. **Cloning a Repository:**
   ```bash
   git clone git clone https://github.com/Putra-Christian618/EcoRouter.git
   cd Project1
2. **Run Container:**
   Run the following command to download all dependencies (Streamlit, PyTorch, OR-Tools) and start the server:
   ```bash
   docker compose up --build
4. **App Access:**
   Wait until the terminal displays the "running" status. Open your web browser and access the following link: [Localhost Aplikasi](http://localhost:8501/)

## Dataset & Model
1. Packet Detection Dataset: The model was fine-tuned locally using the public "Warehouse Delivery Box Detection Dataset," employing aggressive data augmentation (Mosaic, Mixup, HSV) to prevent overfitting to lighting variations.
2. Logistics Dataset: A catalog of weight, volume, and coordinate profiles, simulated using the Olist Brazilian E-Commerce Dataset following rigorous modification and cleaning.
3. AI Model: Pre-trained YOLOv11n, fine-tuned for 100 epochs utilizing CUDA acceleration in a local WSL environment.
