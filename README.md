# Pre-Departure Checklist Detector - YOLO Object Detection

Sistem deteksi objek real-time berbasis YOLO (You Only Look Once) untuk membantu mendeteksi keberadaan barang-barang penting sebelum meninggalkan kost, menggunakan webcam secara langsung melalui browser.

---

## Latar Belakang & Permasalahan

Salah satu masalah sehari-hari yang sering dialami mahasiswa (khususnya anak kost) adalah lupa membawa barang penting saat akan bepergian, seperti HP, botol minum, atau buku. Hal ini sering menyebabkan harus balik lagi ke kost atau kerepotan di tengah jalan.

Proyek ini mencoba menyelesaikan masalah tersebut menggunakan pendekatan Computer Vision, di mana kamera akan otomatis mendeteksi apakah barang-barang penting sudah terlihat/tersedia sebelum pengguna keluar rumah.

---

## Fungsi & Fitur

- Deteksi objek real-time menggunakan webcam langsung dari browser (tanpa perlu install aplikasi tambahan)
- Checklist otomatis - sistem mengecek apakah barang-barang penting yang sudah ditentukan (`checklist_items`) terdeteksi dalam frame
- Visual feedback langsung - bounding box hijau untuk semua objek yang terdeteksi model
- Status indikator per barang:
  - OK - barang terdeteksi di frame
  - MISSING - barang belum/tidak terdeteksi
- Notifikasi ringkasan di console - menyebutkan barang apa saja yang masih hilang, atau konfirmasi jika semua barang sudah lengkap
- Mudah dikustomisasi - daftar barang checklist bisa diganti dengan mudah tanpa mengubah logic inti (cukup ubah 1 baris kode)
- Tanpa training ulang - menggunakan model pretrained `yolo26n.pt` yang sudah dilatih pada dataset COCO (80 kelas objek umum), sehingga bisa langsung dipakai tanpa proses training tambahan

---

## Model & Teknologi

| Komponen | Keterangan |
|---|---|
| Model | YOLO26n (`yolo26n.pt`) - varian nano, ringan dan cepat |
| Framework | Ultralytics (https://github.com/ultralytics/ultralytics) |
| Dataset | COCO Dataset (pretrained, 80 kelas objek umum) |
| Bahasa | Python 3 |
| Library pendukung | OpenCV, NumPy, Pillow, IPython |
| Environment | Google Colab (menggunakan webcam via JavaScript) |
| Kelas COCO yang digunakan | `bottle`, `cell phone`, `book` |

Model ini tidak memerlukan proses training/fine-tuning, karena seluruh objek pada checklist sudah termasuk dalam 80 kelas bawaan COCO dataset yang dikenali oleh model pretrained YOLO26n.

---

## Instalasi

### 1. Clone repository
```bash
git clone https://github.com/username/pre-departure-checklist-yolo.git
cd pre-departure-checklist-yolo
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

Isi `requirements.txt`:
```
ultralytics
opencv-python
numpy
pillow
```

### 3. Jalankan notebook
Buka file `notebook.ipynb` menggunakan Google Colab (direkomendasikan, karena membutuhkan akses webcam via browser):

1. Upload notebook ke Google Colab (https://colab.research.google.com/)
2. Jalankan seluruh cell secara berurutan (Runtime > Run all)
3. Izinkan akses kamera saat diminta oleh browser
4. Sistem akan otomatis mulai mendeteksi barang secara real-time

Catatan: karena menggunakan `getUserMedia()` (akses kamera browser), notebook ini hanya bisa dijalankan di lingkungan yang mendukung interaksi JavaScript seperti Google Colab, bukan di Jupyter Notebook lokal biasa.

---

## Cara Kerja

Sistem berjalan melalui beberapa tahap utama, dengan alur sebagai berikut:

```
1. Ambil Frame Kamera      -> take_photo() + js_to_image()
2. Deteksi Objek YOLO      -> model(img)
3. Cek Checklist           -> check_checklist(results)
4. Gambar Bounding Box     -> draw_detections(img, results)
5. Gambar Status Checklist -> draw_checklist_status(img, detected)
6. Tampilkan Frame + Notifikasi -> display() + print status
   (loop berulang setiap 0.1 detik)
```

### Rincian tiap komponen:

**1. `checklist_items`**
Daftar nama barang yang ingin dicek keberadaannya, harus sesuai dengan nama kelas pada dataset COCO.
```python
checklist_items = ["bottle", "cell phone", "book"]
```

**2. `check_checklist(results)`**
Menerima hasil deteksi YOLO, lalu mengecek apakah label dari setiap objek yang terdeteksi cocok dengan salah satu item di `checklist_items`. Mengembalikan dictionary status (True/False) untuk tiap barang.

**3. `draw_detections(img, results)`**
Menggambar bounding box hijau beserta label dan confidence score di sekitar setiap objek yang berhasil dideteksi oleh model, terlepas dari objek tersebut ada di checklist atau tidak.

**4. `draw_checklist_status(img, detected)`**
Menampilkan status tiap barang checklist di pojok kiri atas layar - teks hijau (OK) jika terdeteksi, teks merah (MISSING) jika belum terdeteksi.

**5. `start_streaming_checklist()`**
Fungsi utama yang menjalankan seluruh alur di atas secara berulang (loop) - mengambil frame baru dari kamera setiap kurang lebih 0.1 detik, sehingga terlihat seperti video real-time.

**6. Eksekusi**
```python
start_streaming_checklist()
```
Loop akan terus berjalan sampai dihentikan manual (interrupt), sehingga cocok digunakan sebagai pengecekan sebelum keluar kost.

---

## Tampilan Hasil

### Kondisi 1 - Semua barang lengkap
(Sisipkan screenshot: semua bounding box hijau, status checklist "OK" di semua item)

Contoh output console:
```
Semua barang lengkap!
```

### Kondisi 2 - Ada barang yang belum terdeteksi
(Sisipkan screenshot: sebagian item berstatus "MISSING" berwarna merah)

Contoh output console:
```
Belum kedeteksi: bottle, book
```

## Kustomisasi

Ingin mengganti barang yang dicek? Cukup ubah isi `checklist_items`, tidak perlu mengubah kode lainnya:

```python
checklist_items = ["cell phone", "laptop", "backpack"]
```

## Lisensi

Proyek ini dibuat untuk keperluan akademik (tugas UAS mata kuliah Multimedia Cerdas).
