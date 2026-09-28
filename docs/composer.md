# Composer

Composer adalah **pengelola paket PHP**. Laravel sendiri dan hampir semua library tambahan (disebut *package*) dipasang lewat Composer. Kamu cukup menyebut nama package, lalu Composer mengunduhnya beserta semua yang dibutuhkan package itu.

## Berkas yang perlu dikenal

| Berkas atau folder | Fungsi |
|---|---|
| `composer.json` | Daftar package yang dibutuhkan project dan aturan versinya. Kamu (atau Composer) yang mengedit. |
| `composer.lock` | Catatan **versi persis** yang terpasang, supaya semua orang mendapat versi yang sama. **Di-commit ke Git.** |
| `vendor/` | Tempat semua package diunduh. **Tidak di-commit** dan tidak boleh diedit manual. Bisa dibuat ulang kapan saja dengan `composer install`. |

## Command dasar

| Command | Penjelasan | Contoh |
|---|---|---|
| `composer install` | Memasang semua package sesuai `composer.lock`. Dipakai setelah meng-clone project atau saat deploy. | `composer install` |
| `composer require` | **Menambah package baru.** Composer mencari versi terbaru yang cocok, memasangnya, lalu mengubah `composer.json` dan `composer.lock`. | `composer require spatie/laravel-permission` |
| `composer require --dev` | Sama seperti di atas, tetapi untuk package yang hanya dibutuhkan saat development (alat debug, test, dst.). | `composer require barryvdh/laravel-debugbar --dev` |
| `composer remove` | Menghapus package. | `composer remove spatie/laravel-permission` |
| `composer update` | Memperbarui semua package ke versi terbaru yang masih diizinkan `composer.json`, lalu memperbarui `composer.lock`. | `composer update` |
| `composer update nama/package` | Memperbarui satu package saja. Lebih aman daripada memperbarui semuanya. | `composer update spatie/laravel-permission` |
| `composer show` | Menampilkan daftar package yang terpasang. | `composer show` |
| `composer outdated` | Menampilkan package yang punya versi lebih baru. | `composer outdated` |
| `composer dump-autoload` | Membuat ulang peta class. Jalankan kalau class baru tidak terdeteksi, misalnya setelah mengubah bagian `autoload` di `composer.json`. | `composer dump-autoload` |
| `composer run` | Menjalankan script yang didefinisikan di `composer.json`. | `composer run dev` |
| `composer global require` | Memasang package secara global untuk seluruh komputer, bukan per project. | `composer global require laravel/installer` |
| `composer self-update` | Memperbarui Composer itu sendiri. | `composer self-update` |

Kalau memakai Herd, `herd composer` menjalankan Composer dengan versi PHP yang dipakai project saat ini.

## Memasang package baru: langkah demi langkah

Kunjungi **packagist.org** untuk mencari package dan membaca petunjuk pemasangannya. Polanya hampir selalu sama:

**1. Pasang dengan `composer require`**
```bash
composer require spatie/laravel-permission
```

**2. Jalankan langkah tambahan dari dokumentasi package**

Banyak package meminta kamu mempublikasikan file konfigurasi atau migrasi. Contoh untuk package di atas:
```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

**3. Pakai package di kode**

Baca dokumentasinya. Contoh: tambahkan trait `HasRoles` pada model `User`.

### Contoh lain: package untuk development

Package yang hanya dipakai saat development dipasang dengan `--dev`, sehingga tidak ikut terpasang di production:
```bash
composer require barryvdh/laravel-debugbar --dev
```

Laravel mendeteksi *service provider* sebagian besar package secara otomatis (*package discovery*), jadi biasanya tidak perlu mendaftarkannya manual.

## Aturan versi

Di `composer.json` kamu akan melihat versi seperti `^2.0`. Tanda `^` artinya "boleh naik ke versi baru selama masih di angka mayor yang sama".

| Penulisan | Artinya |
|---|---|
| `^2.0` | Boleh `2.0` sampai sebelum `3.0` |
| `2.4.1` | Harus persis versi ini |
| `dev-main` | Mengambil langsung dari branch `main` (tidak stabil) |

Untuk memasang versi tertentu: `composer require nama/package:^2.0`.

## Cara aman memakai `install` dan `update`

- **Setelah clone atau saat deploy:** pakai `composer install`, karena ia mengikuti `composer.lock` sehingga versi sama dengan yang sudah teruji.
- **Ingin memperbarui package:** pakai `composer update nama/package`, lalu tes aplikasi sebelum commit `composer.lock`.
- **Untuk production:** `composer install --no-dev --optimize-autoloader`. Opsi `--no-dev` melewati package development, dan `--optimize-autoloader` (singkatnya `-o`) mempercepat pemuatan class.

## Masalah yang sering muncul

| Masalah | Penyebab dan solusi |
|---|---|
| `Your requirements could not be resolved to an installable set of packages` | Package belum mendukung versi Laravel atau PHP yang kamu pakai. Cek alasannya dengan `composer why-not laravel/framework 13.0` atau baca bagian `require` di halaman package. Cari versi package yang lebih baru atau alternatifnya. |
| Peringatan versi PHP tidak cocok | Cek dengan `composer check-platform-reqs` dan `php --version`. Laravel 13 butuh PHP 8.3 atau lebih baru. |
| Kehabisan memori | Jalankan sekali dengan batas memori dimatikan. Bash (Linux, macOS): `COMPOSER_MEMORY_LIMIT=-1 composer require nama/package`. PowerShell: `$env:COMPOSER_MEMORY_LIMIT=-1` lalu jalankan commandnya. |
| Class baru tidak ditemukan | `composer dump-autoload` |
| Composer berperilaku aneh | `composer diagnose` untuk memeriksa masalah umum |
