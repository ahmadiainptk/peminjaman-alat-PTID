# **Dokumen Kebutuhan Bisnis (BRD) & Rancangan Teknis**

## **Aplikasi Peminjaman Alat PTID IAIN Pontianak**

| Proyek | Aplikasi Peminjaman Alat Internal |
| :---- | :---- |
| **Dibuat Untuk** | Pusat Teknologi Informasi dan Data (PTID) IAIN Pontianak |
| **Tanggal** | 5 November 2025 |
| **Versi** | 1.0 |

## **1\. Pendahuluan**

### **1.1. Latar Belakang**

Saat ini, proses peminjaman dan pengembalian alat (seperti laptop, proyektor, kabel, dll.) di lingkungan IAIN Pontianak masih dilakukan secara manual. Hal ini menyebabkan kesulitan dalam pelacakan (tracking) inventaris, memakan waktu administrasi, dan rentan terhadap kehilangan data atau kesalahan pencatatan.

### **1.2. Tujuan Proyek**

Proyek ini bertujuan untuk membangun sebuah aplikasi web terpusat yang men-digitalisasi seluruh alur peminjaman alat. Aplikasi ini akan menjadi sumber data tunggal (single source of truth) untuk status inventaris, riwayat peminjaman, dan data peminjam.

Aplikasi ini dirancang sebagai aplikasi *serverless* yang modern, sangat cepat, dan hemat biaya, dengan memanfaatkan tumpukan teknologi Cloudflare dan Google Workspace yang sudah dimiliki oleh institusi.

## **2\. Ruang Lingkup Proyek**

### **2.1. Dalam Ruang Lingkup (In-Scope)**

* Autentikasi pengguna menggunakan akun Google Workspace (@iain-pontianak.ac.id).  
* Proses peminjaman alat oleh pegawai (via scan QR atau form manual).  
* Proses pengembalian alat.  
* Panel admin untuk mengelola daftar alat (CRUD \- Create, Read, Update, Delete).  
* Panel admin untuk melihat riwayat dan status peminjaman.  
* Pencadangan (arsip) data lama secara otomatis ke Google Workspace (Sheets & Drive).

### **2.2. Luar Ruang Lingkup (Out-of-Scope)**

* Sistem *booking* atau reservasi alat untuk masa depan.  
* Manajemen siklus hidup aset (misal: proses pengadaan, penghitungan depresiasi).  
* Integrasi dengan sistem di luar PTID (misal: sistem keuangan, SDM).  
* Aplikasi seluler (native) \- fokus pada aplikasi web responsif.

## **3\. Aktor dan Peran**

| Aktor | Peran | Hak Akses |
| :---- | :---- | :---- |
| **Peminjam** | Pegawai / Staf IAIN Pontianak | \- Login menggunakan akun Google (@iain-pontianak.ac.id). \- Melihat daftar alat yang tersedia. \- Melakukan peminjaman. \- Melihat riwayat peminjaman pribadi. \- Melakukan pengembalian. |
| **Admin** | Staf PTID | \- Memiliki semua hak akses Peminjam. \- Mengelola (CRUD) data alat, termasuk mengunggah foto. \- Mencetak QR code untuk setiap alat. \- Melihat seluruh riwayat transaksi peminjaman. \- Mengonfirmasi pengembalian alat (jika diperlukan). \- Mengakses laporan arsip di Google Sheets/Drive. |

## **4\. Persyaratan Fungsional (Functional Requirements)**

### **F-1: Modul Autentikasi**

* **F-1.1:** Aplikasi harus menyediakan tombol "Login with Google".  
* **F-1.2:** Sistem *harus* memverifikasi token login Google di sisi *backend* (Cloudflare Worker).  
* **F-1.3:** Sistem *harus* menolak login dari akun email yang **tidak** berdomain @iain-pontianak.ac.id.

### **F-2: Modul Peminjaman (Alur Peminjam)**

* **F-2.1:** Peminjam dapat melihat galeri/daftar alat yang berstatus "Tersedia".  
* **F-2.2:** Peminjam dapat memulai alur peminjaman dengan dua cara:  
  * (a) Memindai (scan) QR code yang tertempel di alat menggunakan kamera HP.  
  * (b) Memilih alat secara manual dari daftar di aplikasi.  
* **F-2.3:** Sistem mencatat transaksi peminjaman (ID Peminjam, ID Alat, Waktu Pinjam) ke database "panas" (Cloudflare D1).  
* **F-2.4:** Status alat berubah menjadi "Dipinjam".

### **F-3: Modul Pengembalian (Alur Peminjam)**

* **F-3.1:** Peminjam dapat melihat daftar alat yang sedang dipinjamnya.  
* **F-3.2:** Peminjam dapat menekan tombol "Kembalikan" pada alat yang ingin dikembalikan.  
* **F-3.3:** (Opsional, ditentukan developer) Pengembalian alat mungkin memerlukan konfirmasi dari Admin PTID untuk mengubah status alat kembali menjadi "Tersedia".

### **F-4: Modul Admin (Manajemen Alat)**

* **F-4.1:** Admin memiliki halaman khusus untuk mengelola inventaris.  
* **F-4.2:** Admin dapat menambahkan, mengedit, dan menghapus data alat (Nama, Deskripsi, Kategori).  
* **F-4.3:** Admin dapat mengunggah foto untuk setiap alat. Foto ini *harus* disimpan di *storage* "panas" (Cloudflare R2).  
* **F-4.4:** Sistem secara otomatis menghasilkan QR code unik untuk setiap alat baru.  
* **F-4.5:** Admin dapat melihat dasbor berisi status semua alat (Tersedia, Dipinjam) dan riwayat lengkap peminjaman.

## **5\. Persyaratan Non-Fungsional (Non-Functional Requirements)**

Ini adalah bagian krusial yang mendefinisikan arsitektur teknis aplikasi.

### **NF-1: Tumpukan Teknologi (Technology Stack) \- WAJIB**

* **Frontend:** **Nuxt.js** (versi terbaru), di-deploy di **Cloudflare Pages**.  
* **Backend API:** **Cloudflare Workers** (terintegrasi dengan server Nitro Nuxt).  
* **Database (Hot Storage):** **Cloudflare D1**. Digunakan untuk data operasional harian (daftar alat, transaksi aktif).  
* **Storage Foto (Hot Storage):** **Cloudflare R2**. Digunakan untuk menyimpan foto alat yang ditampilkan di aplikasi.

### **NF-2: Autentikasi & Otorisasi \- WAJIB**

* **Autentikasi:** Menggunakan **Google Sign-In**. Proses *login* menghasilkan **Google ID Token** (JWT).  
* **Otorisasi:** Token JWT *harus* dikirim ke *backend* (Cloudflare Worker) pada setiap *request* API yang terproteksi. Worker *harus* memverifikasi token ini (menggunakan google-auth-library atau sejenisnya) dan *wajib* memeriksa *payload* email untuk memastikan domain @iain-pontianak.ac.id.

### **NF-3: Strategi Penyimpanan & Arsip \- WAJIB**

Untuk menjaga agar tetap berada dalam batas *free tier* Cloudflare (10GB), strategi *Hybrid Storage* ini *harus* diimplementasikan.

* **Penyimpanan "Panas" (Cloudflare):** D1 dan R2 digunakan untuk data operasional aktif seperti yang dijelaskan di NF-1.  
* **Penyimpanan "Dingin" (Google Workspace):**  
  * **Google Sheets** digunakan sebagai arsip *database* (data transaksi lama).  
  * **Google Drive** digunakan sebagai arsip *file* (foto dari alat yang diarsip).  
* **Proses Arsip Otomatis:**  
  * Sebuah **Cloudflare Cron Trigger** *harus* disetel untuk berjalan secara periodik (misal: sebulan sekali pukul 3 pagi).  
  * Skrip *worker* ini *harus* melakukan:  
    1. Mengautentikasi diri ke Google API menggunakan **Google Service Account** (kunci JSON disimpan sebagai *secret* di Cloudflare).  
    2. Membaca data dari D1 yang lebih lama dari ambang batas (misal: 6 bulan).  
    3. Membaca foto terkait dari R2.  
    4. Menulis data transaksi ke **Google Sheets** (Arsip).  
    5. Mengunggah file foto ke **Google Drive** (Arsip).  
    6. Setelah **terkonfirmasi sukses** tersimpan di Google, skrip *harus* **menghapus** data dan foto lama tersebut dari D1 dan R2.

### **NF-4: Kinerja & Keamanan**

* Aplikasi harus responsif dan dapat diakses di perangkat seluler (Mobile-First).  
* Seluruh komunikasi antara *frontend* dan *backend* harus melalui HTTPS (disediakan oleh Cloudflare).  
* Tidak boleh ada *secret* atau kunci API yang terekspos di sisi *frontend* (Nuxt). Semua harus disimpan di *environment variables* Cloudflare Worker.

## **6\. Asumsi dan Batasan**

* IAIN Pontianak memiliki akun Google Workspace yang aktif.  
* Seluruh Peminjam memiliki akun Google aktif berdomain @iain-pontianak.ac.id.  
* Tim pengembang akan diberi akses ke *dashboard* Cloudflare dan konsol Google Cloud Project untuk mengelola Service Account.  
* Volume peminjaman harian dan pertumbuhan data diperkirakan tidak akan melampaui batas *free tier* Cloudflare D1/R2 (10GB) sebelum proses arsip berjalan.

## **7\. Dokumen Terkait**

* **Diagram Alur Logika: Aplikasi Peminjaman Alat PTID** (diagram\_alur\_aplikasi.html) \- Dokumen ini adalah pelengkap visual untuk BRD ini dan menjelaskan alur data secara rinci.
