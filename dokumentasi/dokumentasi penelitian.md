# Dokumentasi Sistem

## 1. Gambaran Umum
Sistem berupa peta web interaktif untuk menampilkan hasil analisis aksesibilitas dan kesesuaian lokasi SMA/sederajat pada tingkat desa. Wilayah studi: [ISI].

## 2. Arsitektur
| Komponen | Keterangan |
|---|---|
| Pemodelan | [ISI: Python/R/QGIS, pustaka yang dipakai] |
| Data hasil | [ISI: GeoJSON/CSV hasil E2SFCA dan MCDA] |
| Antarmuka | Web statis (HTML/JS/CSS), peta [ISI: Leaflet/MapLibre] |
| Hosting | Vercel (https://web-skripsi-very.vercel.app/) |

Alur: data mentah → preprocessing → pemodelan E2SFCA & MCDA → data hasil (GeoJSON) → peta web.

## 3. Metode
### 3.1 E2SFCA (*Enhanced Two-Step Floating Catchment Area*)
1. Langkah 1: hitung rasio penyedia terhadap populasi di area tangkapan tiap sekolah, dengan bobot peluruhan jarak.
2. Langkah 2: jumlahkan rasio dari semua sekolah dalam jangkauan tiap desa menjadi indeks aksesibilitas.

Parameter: ambang jarak/waktu [ISI], fungsi peluruhan [ISI], ukuran kapasitas [ISI].

### 3.2 MCDA (*Multi-Criteria Decision Analysis*)
Kriteria: [ISI]. Metode pembobotan: [ISI, mis. AHP]. Metode agregasi: [ISI, mis. weighted overlay]. Kelas kesesuaian: [ISI].

### 3.3 Kebijakan Lokasi
Aturan penggabungan hasil aksesibilitas dan kesesuaian menjadi rekomendasi tindakan per desa: [ISI, mis. tabel matriks].

## 4. Fitur Artefak
Tiga lapisan analisis, filter (kabupaten/kota, kecamatan, kategori), pencarian desa/kecamatan, legenda, ringkasan, klik wilayah untuk zoom dan rincian, dan citra satelit.

## 5. Keterbatasan
[ISI]
