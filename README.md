# Ujian Tengah Semester (UTS) - Graf Sains Data
**Program Studi S1 Sains Data — Universitas Negeri Surabaya**

---

### **Identitas Mahasiswa**
* **Nama** : Bima Setia Sugiharto
* **NIM** : 24031554040
* **Kelas** : 2024A
* **Mata Kuliah** : Graf Sains Data (3 SKS)
* **Dosen Pengampu** : Tim Dosen Graf Sains Data

---

## 📌 Ringkasan Proyek

Repositori ini berisi dokumentasi lengkap dan kode implementasi **Ujian Tengah Semester (UTS) Graf Sains Data**. Proyek ini berfokus pada pemodelan graf interaksi e-commerce (*RetailRocket Dataset*) menggunakan **Neo4j Graph Database**, **Vector Space Modeling (VSM)**, proyeksi graf bipartit, serta pembangunan sistem rekomendasi berbasis **User-based Collaborative Filtering (K-Nearest Neighbors / KNN)** yang dievaluasi dengan metrik **Hit@5** dan **NDCG@5**.

---

## 📂 Struktur Repositori

```text
├── Tugas Pendahuluan.ipynb              # Eksplorasi driver Neo4j, Cypher DDL/DML, & batch import
├── UTS GSD_2.1 Wrangling dan EDA.ipynb  # Filtering temporal P1/P2, VSM User/Item, Proyeksi & Similarity
├── UTS GSD_2.2 Analytics.ipynb          # User acak reproducible, KNN (k=5), Hit@5, & NDCG@5
├── requirements.txt                     # Daftar dependensi pustaka Python
├── .gitignore                           # Filter file lokal dan dataset mentah berukuran besar
├── data/
│   ├── events_p1_training.csv           # Dataset interaksi Fold Training P1 (15 Mei – 30 Juni 2015)
│   └── events_p2_testing.csv            # Dataset interaksi Fold Testing P2 (01 Juli – 31 Juli 2015)
└── README.md                            # Dokumentasi utama proyek
```

---

## ⏱️ Penentuan Periode Data (Wrangling & EDA)

Pemisahan data interaksi dilakukan secara temporal (*chronological split*) untuk mencegah kebocoran data (*data leakage / lookahead bias*):

| Fold | Rentang Waktu (Masehi) | UNIX Timestamp (Epoch ms) | Total Baris Asli | Keterangan Metodologis |
| :--- | :--- | :--- | :---: | :--- |
| **P1 (Training)** | **15 Mei 2015 – 30 Juni 2015** | `1431648000000` s/d `1435708799999` | 967.112 baris | Digunakan untuk melatih graf interaksi & matriks VSM. |
| **P2 (Testing)**  | **01 Juli 2015 – 31 Juli 2015** | `1435708800000` s/d `1438387199999` | 697.984 baris | Menjadi *ground truth target* untuk menguji rekomendasi. |

* **Alasan Pemilihan**:
  1. **Siklus Bulanan Penuh**: Fold P2 mencakup satu bulan kalender penuh (Juli) sehingga menangkap pola belanja mingguan (*weekday vs weekend*) secara utuh tanpa bias pecahan hari.
  2. **Keseimbangan Volume Data**: Menghasilkan rasio train-test yang seimbang ($\approx 58\% : 42\%$).
  3. **Mitigasi *Concept Drift***: Jendela training 1,5 bulan menjaga agar model mempelajari preferensi terkini tanpa terdistorsi data usang.

---

## 🔬 Temuan Utama & Analisis

### 1. Komposisi Subgraph & Proyeksi Terbaik (Bagian 2.1)
* **Komposisi Graf Bipartit**: Terdiri dari **3.862 simpul `Visitor`** dan **4.031 simpul `Item`** yang seimbang.
* **Proyeksi Terbaik**: **Item-Item Projection** terbukti jauh lebih efektif dan efisien dibanding User-User karena:
  1. *Kompleksitas Komputasi*: Katalog produk jauh lebih stabil dan berukuran lebih kecil dibanding jumlah pengunjung yang bertambah setiap detik ($N_{user} \gg N_{item}$).
  2. *Stabilitas Model*: Relasi kesamaan antar-produk bersifat jangka panjang, sedangkan preferensi pengunjung cepat berubah (*churn*).
  3. *Mengatasi Sparsity*: Produk populer mengumpulkan interaksi dari banyak user sehingga representasi fiturnya jauh lebih padat.
* **Perhitungan Kesamaan**: Menggunakan *Cosine Similarity*, dihasilkan 200 pasangan relasi `[:SIMILAR {score}]` yang dituliskan kembali ke Neo4j. Pada item acuan `4887`, didapatkan 5 rekomendasi teratas:
  1. Item `146735` (Skor: 0.8476)
  2. Item `259078` (Skor: 0.4162)
  3. Item `240004` (Skor: 0.3615)
  4. Item `7943`   (Skor: 0.1783)
  5. Item `152348` (Skor: 0.1209)

---

### 2. Rekomendasi KNN & Evaluasi Metrik (Bagian 2.2)
* **Pemilihan User Acak (*Reproducible*)**:
  Menggunakan `numpy.random.seed(1143)` pada himpunan pengguna berulang (*common active visitors*), terpilih:
  * **Target User**: `414649`
  * **Riwayat Interaksi P1**: `[330090]`
  * **Interaksi Aktual P2 (*Ground Truth*)**: `[869, 118551, 303828]`
* **Rekomendasi K-Nearest Neighbors ($k = 5$)**:
  * Peringkat 1: Item `234808` (Skor: 0.7071)
  * Peringkat 2: Item `308438` (Skor: 0.5774)
  * Peringkat 3: Item `432478` (Skor: 0.5774)
  * Peringkat 4: Item `869`    (Skor: 0.4472)
* **3 Item yang Diiklankan untuk $k = 5$**:
  1. **Item `234808`** (Skor: 0.7071)
  2. **Item `308438`** (Skor: 0.5774)
  3. **Item `432478`** (Skor: 0.5774)
* **Hasil Evaluasi**:
  * **$\text{Hit@5} = 1.0$ (100% Sukses)**: Produk dengan ID `869` yang direkomendasikan pada Top 5 (peringkat ke-4) terbukti benar-benar dikunjungi oleh pengguna pada periode pengujian P2.
  * **$\text{NDCG@5} = 0.4307$ (43.07%)**: Dihitung dari penalti diskonto peringkat ke-4 ($DCG = 1 / \log_2(5) \approx 0.4307$) terhadap skor ideal ($IDCG = 1.0$), membuktikan bahwa rekomendasi relevan berada di dalam batas Top 5.

---

## 🚀 Panduan Menjalankan Proyek

1. **Clone Repositori**:
   ```bash
   git clone https://github.com/<username-github-anda>/UTS-Graf-Sains-Data.git
   cd UTS-Graf-Sains-Data
   ```

2. **Siapkan Virtual Environment & Install Dependensi**:
   ```bash
   python -m venv myenv
   # Di Windows:
   myenv\Scripts\activate
   # Di Linux/Mac:
   source myenv/bin/activate

   pip install -r requirements.txt
   ```

3. **Database Neo4j (Opsional)**:
   * Jalankan Neo4j Desktop / Server lokal pada `bolt://localhost:7687`.
   * Seluruh notebook telah dilengkapi dengan hasil output dan visualisasi grafik yang tersimpan secara permanen.

4. **Buka Notebook**:
   ```bash
   jupyter notebook
   ```
   Buka dan jalankan file `Tugas Pendahuluan.ipynb`, `UTS GSD_2.1 Wrangling dan EDA.ipynb`, dan `UTS GSD_2.2 Analytics.ipynb`.
