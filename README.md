# UTS_PRAKTIKUM_PPW_12S24036_JOSEF-CHRISTIAN-MARPAUNG

SOAL UJIAN TENGAH SEMESTER PRAKTIKUM (HOTS)
Studi Kasus: Pengembangan Sistem Manajemen Aset dan Peminjaman Inventaris 
Laboratorium Terpadu (DEL Lab Equipment & Asset Tracking System - DEL-LEATS)

Laboratorium Rekayasa Perangkat Lunak dan Jaringan di Institut Teknologi Del mengelola ratusan aset perangkat teknologi berharga tinggi (seperti laptop pengujian performa tinggi, drone observasi, kit sensor IoT, server mini, dan tablet digital). Selama ini, proses inventarisasi dan peminjaman perangkat laboratorium oleh mahasiswa untuk keperluan tugas akhir dan riset masih dilakukan secara manual menggunakan lembaran formulir fisik atau spreadsheet terpisah. Hal ini menimbulkan sejumlah kendala operasional yang serius: (1) Sering terjadi peminjaman ganda (double-booking) karena ketersediaan unit tidak termonitor secara real-time, (2) Riwayat kondisi fisikperangkat tidak terdokumentasi secara visual sehingga saat terjadi kerusakan tidak diketahui pihak yang bertanggung jawab, (3) Pengguna kesulitan  mencari dan memfilter perangkat berdasarkan kategori spesifik karena data tidak terpaginasi dengan baik, dan (4) Formulir peminjaman kerap diisi dengan data asal-asalan tanpa penegakan validasi ketat sehingga batas pengembalian sering terlampaui tanpa perhitungan kompensasi yang jelas.
Untuk memecahkan permasalahan di atas, Anda ditugaskan merancang dan membangun modul inti aplikasi web terintegrasi berbasis pola arsitektur Model-View-Controller (MVC) yang menghubungkan antarmuka pengguna interaktif dengan penyimpanan basis data persisten. Aplikasi web yang Anda kembangkan harus mencakup alur lengkap: mulai dari penyiapan skema tabel basis data, pendaftaran inventaris baru disertai pengunggahan berkas foto fisik perangkat, katalog penjelajahan aset berpaginasi dengan pencarian kata kunci dan multi-filter, hingga transaksi pengajuan peminjaman yang menerapkan validasi aturan bisnis ketat, pencegahan serangan formulir, dan mutasi status perangkat secara konsisten

A. SPESIFIKASI KEBUTUHAN SISTEM & ATURAN PENGEMBANGAN
1. Perancangan Skema Basis Data & Migrations (15 Poin) 
Buat berkas migrasi basis data dengan nama tabel dan atribut yang presisi sebagai berikut:
    • Tabel 'categories': 'id' (Primary Key Auto Increment), 'name' (VARCHAR 100, Unique), 'code' (VARCHAR 10, Unique, misal: 'IOT', 'SRV', 'DRN'), 'created_at', 'updated_at'.
    • Tabel 'equipments': 'id' (Primary Key), 'category_id' (Foreign Key merujuk ke 'categories.id' on-delete restrict), 'asset_code' (VARCHAR 20, Unique, format: EQ-[KODE_KAT]-[NOMOR]), 'name' (VARCHAR 150), 'specification' (TEXT), 'condition' (ENUM: 'baik', 'rusak_ringan', 'rusak_berat', default: 'baik'), 'status' (ENUM: 'tersedia', 'dipinjam', 'dalam_perbaikan', default: 'tersedia'), 'daily_rental_fee' (INTEGER, tarif kompensasi harian dalam Rupiah), 'image_path' (VARCHAR 255, nullable, menyimpan jalur berkas gambar perangkat), timestamps.
    • Tabel 'loan_requests': 'id' (Primary Key), 'equipment_id' (Foreign Key merujuk ke 'equipments.id'), 'borrower_name' (VARCHAR 100), 'borrower_nim' (VARCHAR 10), 'borrower_email' (VARCHAR 100), 'loan_date' (DATE), 'expected_return_date' (DATE), 'purpose' (TEXT), 'status' (ENUM: 'diajukan', 'disetujui', 'ditolak', 'selesai', default: 'diajukan'), 'total_estimated_fee' (INTEGER, hasil kalkulasi otomatis selisih hari dikali tarif harian), timestamps.
    • Siapkan berkas Database Seeder untuk mengisi data awal: minimal 3 kategori ('Server & Cloud', 'Internet of Things', 'Multimedia & Drone') dan minimal 6 data perangkat laboratorium dengan status dan kondisi bervariasi.
2. Arsitektur Model, Controller, dan Routing RESTful (15 Poin)
Terapkan pemisahan tanggung jawab MVC secara disiplin:
    • Model Eloquent: 'Category', 'Equipment', dan 'LoanRequest'. Definisikan properti mass assignment ($fillable) dengan aman, serta relasi Eloquent: Category hasMany Equipment, Equipment belongsTo Category, Equipment hasMany LoanRequest.
    • RESTful Routing & Controller Action pada 'routes/web.php':
        1. GET /equipments -> EquipmentController@index (Katalog berpaginasi, pencarian & filter)
        2. GET /equipments/create -> EquipmentController@create (Tampilan formulir tambah aset)
        3. POST /equipments -> EquipmentController@store (Validasi, upload foto, penyimpanan aset)
        4. GET /equipments/{id} -> EquipmentController@show (Detail spesifikasi & ketersediaan aset)
        5. GET /loans/create -> LoanController@create (Formulir pengajuan peminjaman aset)
        6. POST /loans -> LoanController@store (Validasi aturan bisnis & transaksi peminjaman)
3. Desain Antarmuka Semantik HTML5 & CSS Responsif (20 Poin)
• Gunakan layout induk Blade ('layouts/app.blade.php') dengan penandaan elemen semantik HTML5 (<header>, <nav>, <main>, <section>, <article>, <footer>).
    • Buat tata letak responsif menggunakan Flexbox atau CSS Grid (minimal 3 kolom pada desktop, 2 kolom pada tablet, dan 1 kolom pada ponsel cerdas).
    • Terapkan indikator visual status ketersediaan (Status Badge) yang kontras dan jelas:
        1. 'Tersedia' -> Badge warna Hijau (misal: latar #DCFCE7, teks #166534)
        2. 'Dipinjam' -> Badge warna Kuning/Oranye (misal: latar #FEF9C3, teks #854D0E)
        3. 'Dalam Perbaikan' -> Badge warna Merah (misal: latar #FEE2E2, teks #991B1B)
    • Navigasi bar navigasi utama harus menyajikan tautan aktif menuju Katalog Aset, Tambah Aset, dan Riwayat Peminjaman.
4. Pengelolaan Formulir, Validasi Server-Side, & Upload Berkas (25 Poin)
    • Keamanan Formulir: Wajib menyertakan direktif token proteksi CSRF (@csrf) pada setiap elemen formulir metode POST.
    • Formulir Tambah Aset Baru:
        1. Elemen formulir wajib menyertakan atribut enctype="multipart/form-data".
        2. Validasi Server-side: 'asset_code' (required, unique:equipments), 'name' (required, min:5, max:150), 'category_id' (required exists:categories,id), 'daily_rental_fee' (required, numeric, min:0), 'image' (required, image, mimes:jpeg,png,webp, max:2048 KB).
        3. Berkas gambar wajib disimpan ke direktori storage publik dengan penamaan unik berbasis waktu/hash agar tidak terjadi penimpaan berkas.
    • Formulir Pengajuan Peminjaman:
        1. 'equipment_id': Wajib dipilih dan saat submit harus diverifikasi bahwa statusnya bernilai 'tersedia'. Jika berstatus 'dipinjam' atau 'dalam_perbaikan', sistem wajib menolak dan menampilkan pesan galat bisnis.
        2. 'borrower_name': Required, minimal 3 karakter.
        3. 'borrower_nim': Required, tepat 8 digit angka/karakter format NIM Del.
        4. 'borrower_email': Required, format email valid.
        5. 'loan_date': Required, date, tidak boleh sebelum hari ini (after_or_equal:today).
        6. 'expected_return_date': Required, date, after_or_equal:loan_date, dan durasi sewa maksimal 7 hari kalender.
    • Penanganan Error & Retensi Input: Jika terjadi kegagalan validasi, form tidak boleh dikosongkan. Nilai input yang sudah diketikkan harus dipertahankan menggunakan helper old(), dan pesan galat spesifik wajib muncul tepat di bawah field input yang bermasalah.
    • Post-Redirect-Get (PRG) & Flash Notification: Setelah transaksi berhasil, arahkan pengguna kembali dengan pesan notifikasi kilat (flash session message) berwarna hijau yang muncul di bagian atas halaman.
5. Paginasi Dinamis, Filter Kategori, & Pencarian Kata Kunci (25 Poin)
    • Pada halaman katalog peralatan ('GET /equipments'), tampilkan data secara berpaginasi sebanyak 5 atau 6 kartu aset per halaman.
    • Sediakan formulir pencarian (metode GET) dengan fitur:
        1. Input teks kata kunci (search keyword) yang mencari secara parsial pada 'name' dan 'asset_code'.
        2. Dropdown filter 'category_id' untuk menyaring berdasarkan kategori.
        3. Dropdown filter 'status' untuk menyaring berdasarkan ketersediaan ('semua', 'tersedia', 'dipinjam', 'dalam_perbaikan').
    • Tautan pagination wajib mempertahankan parameter query string pencarian (menggunakan method ->appends(request()->query()) pada objek paginator).

B. CONTOH INPUT DAN OUTPUT SISTEM
Berikut ini adalah skenario kasus uji konkret (positif, negatif, dan pencarian) beserta ilustrasi tampilan antarmuka yang diharapkan:

KASUS UJI 1: PENDAFTARAN INVENTARIS BARU YANG VALID (SKENARIO POSITIF)
    Parameter Input Formulir (Halaman /equipments/create):
            • Kategori Perangkat : Internet of Things (ID: 2)
            • Kode Aset Inventaris : EQ-IOT-005
            • Nama Perangkat : Raspberry Pi 4 Model B (8GB RAM) Starter Kit
            • Spesifikasi Teknis : Quad-core Cortex-A72 @1.5GHz, 8GB LPDDR4, MicroSD 64GB Class 10, Casing 
        Alumunium Aktif Fan, Power Supply Type-C Resmi
            • Kondisi Fisik : Baik
            • Tarif Kompensasi Harian : 15000
            • Berkas Foto Fisik : raspberry_pi4_kit.png (Ukuran: 820 KB, Tipe MIME: image/png)

Output Sistem yang Dihasilkan pada Kasus Uji 1:
    1. Respons HTTP & Pengalihan:
        Status Code: 302 Found -> Dialihkan (Redirect) ke URL: /equipments
    2. Flash Session Notification (Muncul di bagian atas halaman /equipments):
        +----------------------------------------------------------------------------------------------
        --------+
        | [SUKSES] Perangkat 'Raspberry Pi 4 Model B (8GB RAM) Starter Kit' (Kode: EQ-IOT-005) berhasil 
        didaftarkan!|
        +----------------------------------------------------------------------------------------------
        --------+
    3. Verifikasi Data pada Basis Data (Tabel 'equipments'):
        id : 7
        category_id : 2
        asset_code : 'EQ-IOT-005'
        name : 'Raspberry Pi 4 Model B (8GB RAM) Starter Kit'
        condition : 'baik'
        status : 'tersedia'
        daily_rental_fee : 15000
        image_path : 'equipments/1729420000_eq_iot_005.png'
    4. Tampilan Visual pada Katalog Aset:
        Kartu perangkat muncul paling awal dengan foto thumbnail, badge hijau 'Tersedia', label kategori 'Internet of Things', serta tombol aksi aktif '[Ajukan Pinjam]'.

KASUS UJI 2: PENGAJUAN PEMINJAMAN DENGAN DATA TIDAK VALID (SKENARIO NEGATIF)
    Parameter Input Formulir Pengajuan (Halaman /loans/create):
    • Pilihan Perangkat : EQ-SRV-001 - Dell PowerEdge T40 Server (Saat ini sedang berstatus: 'dipinjam')
    • Nama Peminjam : Jo (Panjang hanya 2 karakter -> Tidak memenuhi batas minimum 3 karakter)
    • NIM Mahasiswa : 12S23 (Hanya 5 karakter -> Format NIM Del tidak valid)
    • Surel Mahasiswa : johannes.del.ac.id (Format alamat email tidak valid, kehilangan simbol '@')
    • Tanggal Pinjam : 2026-10-20 (Tanggal lampau terhadap waktu ujian 30 Oktober 2026)
    • Rencana Kembali : 2026-11-20 (Durasi 31 hari -> Melebihi batas maksimal peminjaman 7 hari)12S3101_PPW | halaman 5
    • Tujuan Peminjaman : Praktikum
Output Sistem yang Dihasilkan pada Kasus Uji 2:
    1. Respons HTTP & Pengalihan:
        Status Code: 302 Found -> Dialihkan kembali ke URL: /loans/create (membawa Session $errors dan Input old())
    2. Flash Alert & Penolakan Bisnis:
    +----------------------------------------------------------------------------------------------
    --------+
    | [GAGAL] Pengajuan tidak dapat diproses. Silakan periksa kembali seluruh masukan data Anda. 
    |
    +----------------------------------------------------------------------------------------------
    --------+
    3. Pesan Galat Spesifik di Bawah Masing-masing Elemen Input:
        - Field Perangkat : 'Perangkat Dell PowerEdge T40 Server saat ini sedang dipinjam dan tidak tersedia.'
        - Field Nama Peminjam : 'Nama peminjam wajib diisi minimal 3 karakter.'
        - Field NIM Mahasiswa : 'Format NIM harus terdiri dari tepat 8 digit angka/huruf institusi.'
        - Field Surel Mahasiswa : 'Format alamat surel yang Anda masukkan tidak valid.'
        - Field Tanggal Pinjam : 'Tanggal peminjaman tidak boleh sebelum tanggal hari ini.'
        - Field Tanggal Kembali : 'Durasi peminjaman maksimal adalah 7 hari kalender.'
    4. Retensi Nilai Input Form (Form State Retention):
        Kolom input teks tidak dikosongkan. Nilai 'Jo', '12S23', dan 'johannes.del.ac.id' tetap terisi 
        pada kotak teks.

KASUS UJI 3: PENCARIAN & PEMFILTERAN KATALOG ASET (HTTP GET QUERY STRING)
    Parameter Permintaan HTTP GET melalui URL Browser:
    URL Request : /equipments?search=drone&category_id=3&status=tersedia&page=1
    Penjelasan : Menelusuri seluruh aset yang mengandung kata 'drone', berada pada Kategori ID 3 (Multimedia & 
    Drone), dan hanya berstatus 'tersedia'.
Output Sistem yang Dihasilkan pada Kasus Uji 3:
    1. Tampilan Hasil Filter pada Halaman Web:
    Keterangan Ringkasan: 'Menampilkan 2 dari 2 hasil pencarian untuk kata kunci "drone"'
    2. Konten Kartu Aset yang Dirender:
        [KARTU 1]
        Kode Aset : EQ-DRN-001
        Nama : DJI Mavic Air 2 Fly More Combo
        Kategori : Multimedia & Drone | Status: [TERSEDIA (Hijau)] | Biaya: Rp 50.000 / hari
        Aksi : [Detail Spesifikasi] [Ajukan Peminjaman]
        [KARTU 2]
        Kode Aset : EQ-DRN-003
        Nama : FPV Racing Drone BetaFPV Cetus Pro
        Kategori : Multimedia & Drone | Status: [TERSEDIA (Hijau)] | Biaya: Rp 25.000 / hari
        Aksi : [Detail Spesifikasi] [Ajukan Peminjaman]
    3. Tautan Paginasi (Pagination Links):
        Link navigasi halaman otomatis menyertakan parameter pencarian URL:
        href="/equipments?search=drone&category_id=3&status=tersedia&page=1"

D. RUBRIK PENILAIAN ASESMEN PRAKTIKUM (TOTAL 100 POIN)
No Aspek Penilaian & Komponen Fungsional Bobot
    1 Perancangan Skema Basis Data, Migrations, dan Seeders:
    Kelengkapan tabel categories, equipments, loan_requests, relasi foreign key, tipe data yang tepat, unique constraint, serta seeder minimal 3 kategori. 15 Poin
    2 Arsitektur Model, Controller, dan Routing RESTful:
    Penerapan disiplin MVC, Route definitions yang semantik, deklarasi $fillable, pemetaan relasi Eloquent, serta struktur metode Controller yang rapi dan terorganisir. 15 Poin
    3 Antarmuka Semantik HTML5, Blade Layouts, & CSS Responsif:
    Penerapan master layout (@extends, @section), elemen semantik HTML5, Flexbox/Grid responsif, kartu aset informatif, serta status badge visual yang kontras dan jelas. 20 Poin
    4 Pengelolaan Formulir, Validasi Server-Side, & Upload Berkas:
    Proteksi CSRF (@csrf), upload berkas gambar (enctype multipart) ke storage dengan nama unik, validasi komprehensif, retensi input form via old(), dan notifikasi flash session. 25 Poin
    5 Paginasi Dinamis, Filter Kategori, & Pencarian Kata Kunci:
    Implementasi paginasi Eloquent (5 data per halaman), pencarian fleksibel nama/kode aset, filter kategori & status, serta penautan query string URL yang konsisten pada link pagination. 25 Poin

E. PROSEDUR PENGUMPULAN & INSTRUKSI AKHIR
    1. Seluruh kode sumber proyek wajib dikelola dalam repositori Git lokal dengan branch utama 'main'.
    2. Pastikan file konfigurasi '.env.example' telah diperbarui dan perintah migrasi/seeder ('php artisan migrate --seed') dapat dieksekusi dengan lancar tanpa error.
    3. Lakukan kompresi direktori proyek ke dalam berkas archive ZIP dengan format nama: 'UTS_PRAKTIKUM_PPW_[NIM]_[NAMA_LENGKAP].zip' (Kecualikan direktori 'vendor/' dan 'node_modules/' untuk menghemat ukuran berkas).
    4. Unggah berkas ZIP tersebut ke platform e-Course Institut Teknologi Del sebelum batas waktu 240 menit berakhir.
    5. Keterlambatan pengumpulan berkas akan dikenakan penalti pemotongan nilai sebesar 10 poin per 15 menit.
    6. Buat video rekaman di akhir sesi (Setelah ujian) berdurasi maksimal 20 menit yang menjelaskan kode program beserta output kompilasi sistem anda. Kemudian upload ke Youtube pribadi anda(visibility: public). Berikan penamaan: Nim_Nama_UTS_Praktikum_PPW_2026. Upload link Github dan YT ke ecourse.