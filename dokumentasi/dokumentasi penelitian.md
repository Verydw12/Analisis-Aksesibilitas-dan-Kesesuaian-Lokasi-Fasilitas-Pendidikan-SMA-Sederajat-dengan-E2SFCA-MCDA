# Dokumentasi Sistem

**Analisis Aksesibilitas dan Kesesuaian Lokasi Fasilitas Pendidikan Jenjang SMA/Sederajat dengan E2SFCA dan MCDA: Studi Kasus Provinsi Daerah Istimewa Yogyakarta**

Penulis: Very Dwi Prasetya (222212907) | Pembimbing: Dr. Rindang Bangun Prasetyo, S.S.T., M.Si.

## 1. Gambaran Umum

### 1.1 Latar Belakang Singkat
Wajib belajar 12 tahun belum sepenuhnya tercapai. Angka Partisipasi Sekolah (APS) usia 16–18 tahun secara nasional hanya 77,86%, lebih rendah daripada usia 13–15 tahun (96,30%) dan 7–12 tahun (99,23%), sehingga transisi ke jenjang SMA/sederajat menjadi titik lemah. Di Provinsi Daerah Istimewa Yogyakarta (DIY), meskipun APS usia 16–18 tahun tertinggi secara nasional (93,03%), rata-rata lama sekolah antarwilayah masih timpang: Gunungkidul 7,64 tahun, Kulon Progo 9,33 tahun, Bantul 10,11 tahun, Sleman 11,42 tahun, dan Kota Yogyakarta 12,13 tahun.

Penelitian terdahulu umumnya hanya menganalisis satu aspek, yaitu aksesibilitas atau kesesuaian lokasi. Penelitian ini menggabungkan keduanya melalui *overlay* indeks aksesibilitas (E2SFCA) dan indeks kesesuaian lokasi (MCDA) untuk menghasilkan arahan kebijakan yang spesifik per desa/kelurahan.

### 1.2 Tujuan
1. Mengukur aksesibilitas fasilitas pendidikan SMA/sederajat pada setiap desa/kelurahan di DIY.
2. Mengukur kesesuaian lokasi pembangunan fasilitas pendidikan SMA/sederajat pada setiap desa/kelurahan di DIY.
3. Menganalisis hubungan keduanya untuk menentukan prioritas kebijakan pembangunan fasilitas pendidikan.

### 1.3 Ruang Lingkup dan Batasan
- Wilayah studi: Provinsi DIY, dengan 439 desa/kelurahan sebagai unit analisis. Analisis kesesuaian dilakukan pada *grid* 500 m × 500 m, lalu diagregasi ke tingkat desa dengan rata-rata tertimbang luas (*area-weighted average*).
- Fasilitas: SMA, SMK, dan MA, baik negeri maupun swasta (469 sekolah).
- Waktu tempuh dihitung dengan OpenRouteService (ORS) moda mobil karena ORS tidak menyediakan moda sepeda motor. Hasilnya dinilai mendekati waktu tempuh sepeda motor pada rute Google Maps.
- Data penduduk usia 16–18 tahun tidak tersedia pada level desa, sehingga permintaan layanan didekati dengan jumlah penduduk total per desa.
- Data pendidikan yang dipakai adalah data resmi tahun 2025.
- Sekolah dan wilayah di luar DIY tidak diikutkan meskipun terjangkau.

### 1.4 Kerangka Kerja
Penelitian mengikuti kerangka CRISP-DM (*business understanding*, *data understanding*, *data preparation*, *modeling*, *evaluation*, *deployment*) yang bersifat iteratif.

## 2. Arsitektur dan Alur Data

| Komponen | Keterangan |
|---|---|
| Pengumpulan data | Dapodik/Disdikpora DIY, BPS, OpenStreetMap, WorldPop, Landsat-8, NASA SRTM/DEMNAS, POI hasil *scraping* |
| Pemrosesan | Python, QGIS, Google Earth Engine (citra Landsat-8) |
| Waktu tempuh | Matriks ORS (*OpenRouteService Tools*) melalui QGIS |
| Pemodelan | E2SFCA, Global Moran's I, LISA, Entropi, WSM, WPM, TOPSIS, *overlay* |
| Antarmuka | *Dashboard* web statis (HTML, CSS, JavaScript) |
| Hosting | Vercel: https://web-skripsi-very.vercel.app/ |

```
Data mentah → Prapemrosesan → E2SFCA (indeks aksesibilitas per desa)
                            → MCDA (skor kesesuaian per grid → agregasi ke desa)
           → Overlay matriks 5×5 → Arahan kebijakan → Evaluasi → Dashboard
```

## 3. Data

### 3.1 Data Aksesibilitas (E2SFCA)

| No | Variabel | Sumber | Fungsi dalam model |
|---|---|---|---|
| 1 | Waktu tempuh desa–sekolah | ORS | Penentuan zona daerah tangkapan |
| 2 | Populasi penduduk | BPS | Sisi permintaan (*demand*) |
| 3 | Jumlah siswa sekolah | Disdikpora DIY | Sisi penawaran (*supply*) |
| 4 | Titik SMA/SMK/MA | Disdikpora DIY | Lokasi sekolah |

### 3.2 Data Kesesuaian Lokasi (MCDA)

| No | Variabel | Sumber | Tipe |
|---|---|---|---|
| 1 | Kepadatan jaringan jalan | OpenStreetMap | *Benefit* |
| 2 | Kepadatan penduduk | WorldPop | *Benefit* |
| 3 | NDVI | Landsat-8 | *Benefit* |
| 4 | Kemiringan lereng | NASA SRTM/DEMNAS | *Cost* |
| 5 | Ketinggian | NASA SRTM/DEMNAS | *Cost* |
| 6 | Jarak kebisingan | POI | *Benefit* |
| 7 | Jarak sungai | OpenStreetMap | *Benefit* |

Variabel *benefit* bernilai makin baik jika makin besar, sedangkan variabel *cost* makin baik jika makin kecil. Pemilihan variabel kesesuaian mengacu pada Tebbi dkk. (2025), Meng dkk. (2025), dan Permendikbudristek No. 22 Tahun 2023 tentang standar sarana prasarana (lokasi aman, nyaman, terhindar dari bahaya dan kebisingan).

## 4. Prapemrosesan Data
- **Data sekolah:** pemeriksaan *missing value* dan duplikasi; koordinat kosong dilengkapi manual melalui Google Maps; data duplikat dihapus; jumlah siswa kosong dilengkapi dari Dapodik.
- **Citra Landsat-8:** diproses di Google Earth Engine dengan *cloud masking*, pemilihan citra bertutupan awan rendah, komposit tahunan menggunakan median, dan pemotongan sesuai batas wilayah studi.
- **Kemiringan dan ketinggian:** diturunkan dari NASA SRTM/DEMNAS beresolusi 30 m.
- **Data POI hasil *scraping*:** *text cleaning*, ekstraksi koordinat dari URL dengan *Regular Expression*, penyaringan berdasarkan *bounding box* dan poligon batas DIY, deduplikasi berbasis URL unik, serta normalisasi atribut.
- **Grid MCDA:** tiap variabel diekstraksi ke *grid* 500 m berproyeksi UTM; hanya *grid* yang bersinggungan dengan batas wilayah yang dipertahankan.
- **Nilai kosong:** populasi diisi nol (area tak berpenghuni); variabel biofisik diimputasi dengan rata-rata.
- **Transformasi:** *clipping* pada persentil ke-1 dan ke-99, transformasi log1p untuk variabel yang sangat menceng, dan normalisasi *Min-Max* ke rentang 0,01–1,00.

## 5. Metode

### 5.1 Indeks Aksesibilitas Spasial (E2SFCA)
*Centroid* desa dipakai sebagai representasi pusat aktivitas penduduk. Waktu tempuh *centroid* ke tiap sekolah dihitung dengan matriks ORS (dibatasi 4.000 permintaan per perhitungan, sehingga dijalankan untuk enam desa terhadap seluruh sekolah). Daerah tangkapan dibagi tiga zona dengan bobot peluruhan yang berkurang setengahnya tiap kenaikan interval 10 menit:

| Zona waktu tempuh | Bobot (W_r) |
|---|---|
| 0–10 menit | 1 |
| 10–20 menit | 0,5 |
| 20–30 menit | 0,25 |

**Langkah 1:** rasio kapasitas sekolah terhadap populasi di daerah tangkapannya.

```math
R_j = \frac{S_j}{\sum_{k \in \{d_{kj} \in D_r\}} P_k W_r}
```

**Langkah 2:** jumlahkan rasio seluruh sekolah yang terjangkau dari desa *i* dengan bobot zona.

```math
A_i = \sum_{j \in \{d_{ij} \in D_r\}} R_j W_r
```

Keterangan: S_j kapasitas siswa sekolah *j*; P_k jumlah penduduk desa *k*; d_kj waktu tempuh; D_r zona waktu ke-*r*; W_r bobot zona. Indeks A_i dinormalisasi ke rentang 0–1 dan diklasifikasikan ke lima kelas dengan *Natural Breaks* (Jenks).

### 5.2 Autokorelasi Spasial
- **Global Moran's I** menguji apakah aksesibilitas membentuk pola mengelompok (H0: I = 0, ditolak jika *p-value* < 0,05).
- **LISA** (*Local Indicators of Spatial Association*) memetakan klaster *High-High* (*hotspot*), *Low-Low* (*coldspot*), serta pencilan *High-Low* dan *Low-High*.

### 5.3 Kesesuaian Lokasi (MCDA)
Tujuh variabel kesesuaian dipetakan pada *grid* 500 m × 500 m. Bobot kriteria ditentukan secara objektif dengan **Entropi**: kriteria dengan variasi nilai lebih besar memuat informasi lebih banyak sehingga berbobot lebih besar.

```math
e_j = -\frac{1}{\ln m}\sum_{i=1}^{m} p_{ij}\ln p_{ij}, \qquad w_j = \frac{1-e_j}{\sum_{j=1}^{n}(1-e_j)}
```

Skor kesesuaian dihitung dengan tiga metode:

```math
\mathrm{WSM}(A_i)=\sum_{j=1}^{n} w_j a_{ij}, \qquad \mathrm{WPM}(A_i)=\prod_{j=1}^{n} a_{ij}^{w_j}, \qquad P_i=\frac{S_i^-}{S_i^+ + S_i^-}
```

- **WSM:** menjumlahkan nilai kriteria yang dikalikan bobotnya.
- **WPM:** mengalikan nilai kriteria yang dipangkatkan bobotnya.
- **TOPSIS:** menilai kedekatan alternatif dengan solusi ideal positif dan jaraknya dari solusi ideal negatif.

**Pemilihan metode terbaik** memakai AUC kurva ROC. *Grid* yang memuat minimal satu sekolah eksisting diberi label 1 dan lainnya label 0, lalu skor kesesuaian dipakai sebagai nilai prediksi. Skor terpilih diklasifikasikan ke lima kelas dengan *Natural Breaks* dan diagregasi ke tingkat desa.

### 5.4 *Overlay* dan Arahan Kebijakan
Kelas aksesibilitas dan kelas kesesuaian (masing-masing lima kelas) digabungkan berdasarkan kode desa dalam matriks silang 5 × 5. Ke-25 kombinasi dipetakan ke lima arahan kebijakan:

| Arahan Kebijakan | Kesesuaian | Aksesibilitas | Keterangan |
|---|---|---|---|
| 1. Pembangunan sekolah baru | Tinggi–sangat tinggi | Rendah–sedang | Lahan layak, akses minim |
| 2. Intervensi transportasi & zonasi | Rendah–sedang | Rendah | Lahan kurang layak |
| 3. Peningkatan kapasitas eksisting | Tinggi–sangat tinggi | Tinggi–sangat tinggi | Optimal di kedua sisi |
| 4. Intervensi terpadu | Kombinasi menengah | Sedang–tinggi | Zona transisi |
| 5. Kawasan pemeliharaan | Rendah–sangat rendah | Tinggi–sangat tinggi | Layanan sudah memadai |

## 6. Evaluasi dan Validasi
- Indeks aksesibilitas: korelasi Spearman terhadap rata-rata jarak tempuh desa ke sekolah.
- Konsistensi WSM, WPM, dan TOPSIS: korelasi peringkat Spearman antarmetode.
- Realisme peta kesesuaian: sebaran sekolah eksisting dan peta RTRW.
- Seluruh hasil (aksesibilitas, kesesuaian, arahan kebijakan): validasi visual kondisi lapangan melalui Google Maps.

## 7. Ringkasan Hasil

### 7.1 Aksesibilitas
| Parameter | Nilai |
|---|---|
| Rata-rata / median rasio *supply-demand* | 0,000821 / 0,000403 |
| Rata-rata / median indeks | 0,432 / 0,383 |
| Standar deviasi indeks | 0,266 |
| Global Moran's I (*p-value*) | 0,9149 (0,001) |
| Desa *High-High* / *Low-Low* | 104 / 128 |
| Desa tidak signifikan | 207 |
| Korelasi Spearman dengan rata-rata jarak | −0,86 |

Aksesibilitas berpola pusat–pinggiran. Kelas tinggi dan sangat tinggi terpusat di Kota Yogyakarta, Sleman bagian tengah, dan Bantul bagian utara; kelas rendah dan sangat rendah mendominasi Gunungkidul dan Kulon Progo. Tidak ditemukan pencilan *High-Low* maupun *Low-High*.

### 7.2 Kesesuaian Lokasi
Bobot Entropi kriteria:

| Variabel | Tipe | Entropi (Ej) | Bobot (Wj) |
|---|---|---|---|
| Kepadatan penduduk | *Benefit* | 0,975778 | 0,29144 |
| Ketinggian | *Cost* | 0,983425 | 0,19943 |
| Kemiringan lereng | *Cost* | 0,986179 | 0,16629 |
| Kepadatan jaringan jalan | *Benefit* | 0,987696 | 0,14249 |
| Jarak sungai | *Benefit* | 0,991587 | 0,10122 |
| Jarak kebisingan | *Benefit* | 0,995919 | 0,04911 |
| NDVI | *Benefit* | 0,996303 | 0,04448 |

| Metode | AUC | Korelasi Spearman antarmetode |
|---|---|---|
| TOPSIS (terbaik) | 0,850 | WSM–TOPSIS 0,9891 |
| WSM | 0,831 | WSM–WPM 0,9491 |
| WPM | 0,791 | WPM–TOPSIS 0,9481 |

Sebanyak 71,6% sekolah eksisting berada di kawasan berkesesuaian tinggi dan sangat tinggi, dan 13,6% di kawasan rendah dan sangat rendah.

### 7.3 Arahan Kebijakan
| Arahan Kebijakan | Jumlah Desa | Persentase |
|---|---|---|
| 1. Pembangunan sekolah baru | 31 | 7,1% |
| 2. Intervensi transportasi & zonasi | 162 | 36,9% |
| 3. Peningkatan kapasitas eksisting | 54 | 12,3% |
| 4. Intervensi terpadu | 129 | 29,4% |
| 5. Kawasan pemeliharaan | 63 | 14,4% |
| **Jumlah** | **439** | **100,0%** |

Dari 31 desa pada Kebijakan 1 (Kulon Progo dan Bantul), 16 desa belum memiliki SMA/sederajat sama sekali: Kranggan, Nomporejo, Tayuban, Depok, Panjatan, Ngestiharjo, Kulwaru, Sogan, Kedundang, Demen, Kalidengen, dan Janten (Kulon Progo); Tirtosari, Gadingharjo, Panjangrejo, dan Canden (Bantul).

## 8. *Dashboard* Interaktif
*Dashboard* web (https://web-skripsi-very.vercel.app/) menyajikan tiga lapisan: Aksesibilitas (E2SFCA), Kesesuaian Lahan, dan Kebijakan Lokasi SMA.

Fitur: nama desa, kecamatan, dan kabupaten; nilai indeks, kategori, dan arahan kebijakan; filter menurut kabupaten/kota, kecamatan, dan kategori; pencarian desa/kecamatan; legenda; panel ringkasan; klik wilayah untuk *zoom* dan rincian; citra satelit; tampilan responsif untuk perangkat seluler.

Peta disusun di QGIS, lalu diintegrasikan ke *dashboard* web yang dibangun dengan Python, HTML, CSS, dan JavaScript.

## 9. Keterbatasan
1. Permintaan layanan memakai jumlah penduduk total, bukan penduduk usia 16–18 tahun, sehingga berpotensi bias.
2. Waktu tempuh memakai moda mobil (ORS) sebagai pendekatan sepeda motor.
3. Transportasi publik (Trans Jogja, angkutan perdesaan) belum dimodelkan.
4. Bobot kriteria hanya memakai metode Entropi.
5. Wilayah di luar DIY tidak diikutkan (efek batas wilayah).
6. Variabel nonspasial (sosial ekonomi, biaya pendidikan, preferensi jalur SMA/SMK/MA) belum dimasukkan.
7. Validasi kondisi lapangan dilakukan visual lewat Google Maps, bukan survei langsung.

## 10. Pengembangan Selanjutnya
Menggunakan data penduduk usia 16–18 tahun dari Dukcapil/BPS, memodelkan waktu tempuh sepeda motor, memperluas *buffer* analisis lintas provinsi, mencoba metode pembobotan lain, menambahkan transportasi publik, dan memasukkan variabel nonspasial.

## 11. Referensi Utama
- Luo, W., & Qi, Y. (2009). An enhanced two-step floating catchment area (E2SFCA) method for measuring spatial accessibility to primary care physicians. *Health & Place*, 15(4), 1131–1137.
- Meng, Y. dkk. (2025). Enhancing educational equity and sustainability: A fuzzy-based framework for optimal school site selection. *International Journal of Digital Earth*, 18(1). doi: 10.1080/17538947.2025.2495735
- Tebbi, H., Dridi, H., & Bouder, A. (2025). Analytic hierarchy process vs. machine learning: Mapping school suitability in Batna City, Algeria. *Bulletin of the Serbian Geographical Society*, 105(2), 23–38.
- Sakti, A. D. dkk. (2022). School location analysis by integrating the accessibility, natural and biological hazards. *ISPRS International Journal of Geo-Information*, 11(1), 12. doi: 10.3390/ijgi11010012
- Gufron, F. M., & Pramana, S. (2026). Spatial accessibility index and health facility location suitability based on POI and machine learning in DIY Province. *Inferensi*, 9(2), 217–228.
- Martínez-Plumed, F. dkk. (2021). CRISP-DM twenty years later. *IEEE Transactions on Knowledge and Data Engineering*, 33(8), 3048–3061.
- Permendikbudristek No. 22 Tahun 2023 tentang Standar Sarana dan Prasarana pada PAUD, Jenjang Pendidikan Dasar, dan Menengah.

Daftar pustaka lengkap ada pada makalah seminar proposal skripsi.
