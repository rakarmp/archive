+++
title = 'Dasar Protect Php Web'
description = "Improve Website Php Kamu!"
date = 2025-03-16T13:52:03+07:00
aliases = ["about-us","about-hugo","contact"]
author = "Rakarmp"
+++

Mungkin kamu pernah denger ada web yang banyak diserang Ddos, SQL Injection, Dll. Nah
itu semua sebenarnya bisa diantisipasi dan memang tidak 100% dapat mencegahnya, ini hanya
sekedar antisipasi pertama saja, sini mimin kasih tau yaa.

## Ddos

Mungkin kamu udah ngga asing sama yang satu ini, Ya ini DDoS atau (Distributed Denial of Service) adalah serangan cyber yang bertujuan untuk membuat layanan online tidak tersedia bagi pengguna dengan membanjiri server, jaringan, atau situs web dengan lalu lintas internet yang sangat besar dari berbagai sumber. 

## SQL Injection

SQL Injection adalah teknik serangan siber yang memanfaatkan celah keamanan dalam aplikasi web yang berinteraksi dengan basis data SQL. Penyerang menyisipkan kode SQL berbahaya ke dalam input aplikasi, yang kemudian dieksekusi oleh basis data. Hal ini dapat memungkinkan penyerang untuk:

- Mengakses data sensitif: Penyerang dapat mencuri informasi pribadi pengguna, data keuangan, atau informasi rahasia lainnya.
- Memodifikasi data: Penyerang dapat mengubah, menambahkan, atau menghapus data dalam basis data.
- Mengambil kendali atas server: Dalam kasus yang parah, penyerang dapat mengambil alih kendali penuh atas server basis data.

Untuk Php mungkin saya bisa mencontohkan pencegahannya dengan kode berikut:

```php
<?php

class AntiSerangan {

    private $db;

    // Konstruktor untuk menginisialisasi koneksi database
    public function __construct($db) {
        $this->db = $db;
    }

    // Fungsi untuk mendeteksi pola DDoS sederhana
    public function deteksiDDoS($ip, $maxRequests = 100, $timeFrame = 60) {
        // Catat permintaan dari IP ke dalam tabel request_log
        $stmt = $this->db->prepare("INSERT INTO request_log (ip, timestamp) VALUES (?, ?)");
        $stmt->execute([$ip, time()]);

        // Hitung jumlah permintaan dari IP dalam jangka waktu tertentu
        $stmt = $this->db->prepare("SELECT COUNT(*) FROM request_log WHERE ip = ? AND timestamp > ?");
        $stmt->execute([$ip, time() - $timeFrame]);
        $count = $stmt->fetchColumn();

        // Jika melebihi batas, blokir akses
        if ($count > $maxRequests) {
            http_response_code(429); // Too Many Requests
            die("Terlalu banyak permintaan. Silakan coba lagi nanti.");
        }
    }

    // Fungsi untuk menjalankan query database dengan aman (anti SQL Injection)
    public function queryAman($sql, $params = []) {
        $stmt = $this->db->prepare($sql);
        $stmt->execute($params);
        return $stmt;
    }

    // Fungsi untuk sanitasi input pengguna (anti XSS)
    public function sanitasiInput($input) {
        return htmlspecialchars(strip_tags($input), ENT_QUOTES, 'UTF-8');
    }

    // Fungsi untuk menambahkan header keamanan
    public function tambahHeaderKeamanan() {
        header("X-Content-Type-Options: nosniff"); // Mencegah MIME-type sniffing
        header("X-Frame-Options: DENY"); // Mencegah clickjacking
        header("X-XSS-Protection: 1; mode=block"); // Mengaktifkan filter XSS di browser
        header("Content-Security-Policy: default-src 'self'"); // Membatasi sumber konten
    }
}

// Contoh Penggunaan
try {
    // Koneksi ke database SQLite
    $db = new PDO('sqlite:database.db');
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    // Buat tabel request_log jika belum ada
    $db->exec("CREATE TABLE IF NOT EXISTS request_log (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        ip TEXT NOT NULL,
        timestamp INTEGER NOT NULL
    )");

    // Inisialisasi kelas AntiSerangan
    $antiSerangan = new AntiSerangan($db);

    // Deteksi DDoS berdasarkan IP pengguna
    $ip = $_SERVER['REMOTE_ADDR'];
    $antiSerangan->deteksiDDoS($ip);

    // Tambahkan header keamanan
    $antiSerangan->tambahHeaderKeamanan();

    // Contoh penggunaan query aman dan sanitasi input
    if (isset($_POST['username']) && isset($_POST['password'])) {
        $username = $antiSerangan->sanitasiInput($_POST['username']);
        $password = $antiSerangan->sanitasiInput($_POST['password']);
        
        // Query aman untuk memeriksa login
        $stmt = $antiSerangan->queryAman(
            "SELECT * FROM users WHERE username = ? AND password = ?",
            [$username, $password]
        );
        
        $user = $stmt->fetch(PDO::FETCH_ASSOC);
        if ($user) {
            echo "Login berhasil!";
        } else {
            echo "Username atau password salah.";
        }
    }

} catch (PDOException $e) {
    die("Koneksi database gagal: " . $e->getMessage());
}

?>
```

### Penjelasan Kode

#### 1. Perlindungan DDoS

Fungsi: deteksiDDoS

Cara Kerja:
- Mencatat setiap permintaan dari IP pengguna ke dalam tabel request_log di database SQLite bersama dengan waktu (timestamp).
- Memeriksa jumlah permintaan dari IP tersebut dalam jangka waktu tertentu (default: 60 detik).
- Jika jumlah permintaan melebihi batas (default: 100), server mengembalikan kode status HTTP 429 ("Too Many Requests") dan menghentikan eksekusi.

__Catatan: Ini adalah pendekatan sederhana. Untuk perlindungan DDoS yang lebih kuat, gunakan layanan seperti Cloudflare atau konfigurasi firewall di tingkat server.__

### 2.Perlindungan SQL Injection

Fungsi: queryAman

Cara Kerja:
- Menggunakan prepared statements dengan PDO untuk memastikan input pengguna tidak dapat memanipulasi query SQL.
- Parameter query dilewatkan secara terpisah, sehingga kode berbahaya tidak akan dieksekusi.

__Contoh: Query seperti SELECT * FROM users WHERE username = ? aman dari injeksi karena PDO menangani escaping secara otomatis.__

### 3. Sanitasi Input (Anti XSS)

Fungsi: sanitasiInput

Cara Kerja:
- Membersihkan input pengguna dengan menghapus tag HTML (strip_tags) dan mengonversi karakter khusus menjadi entitas HTML (htmlspecialchars).
- Mencegah serangan Cross-Site Scripting (XSS) dengan memastikan input tidak dieksekusi sebagai kode di browser.

### 4. Header Keamanan

Fungsi: tambahHeaderKeamanan

Cara Kerja:
- Menambahkan header HTTP untuk meningkatkan keamanan:
- X-Content-Type-Options: nosniff: Mencegah browser menebak tipe konten.
- X-Frame-Options: DENY: Mencegah halaman dimuat dalam iframe (anti-clickjacking).
- X-XSS-Protection: Mengaktifkan filter XSS bawaan browser.
- Content-Security-Policy: Membatasi sumber konten yang diizinkan.

## Catatan Penting

- Batasan DDoS: Pendekatan ini hanya deteksi dasar. Untuk perlindungan nyata terhadap DDoS, gunakan solusi infrastruktur seperti load balancer atau layanan pihak ketiga.
- Database: Kode ini menggunakan SQLite untuk kemudahan. Untuk aplikasi produksi, Anda mungkin ingin beralih ke MySQL atau PostgreSQL.
- Keamanan Tambahan: Tambahkan fitur seperti CSRF tokens, enkripsi password (misalnya dengan password_hash), dan logging aktivitas mencurigakan untuk perlindungan lebih lengkap.

__Mungkin Segitu Aja Yaa, See u Next Post__