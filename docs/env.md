# File `.env`

File `.env` adalah tempat menyimpan **pengaturan yang berbeda di tiap komputer atau server**, misalnya nama database, password, dan kunci API. Dengan memisahkannya dari kode, project yang sama bisa berjalan di laptop kamu, server testing, dan server production dengan pengaturan masing-masing tanpa mengubah kode.

## Aturan dasar

- File `.env` ada di **root project** (sejajar dengan `composer.json`). Namanya diawali titik, jadi di Linux dan macOS ia tersembunyi. Tampilkan dengan `ls -a`.
- `.env` **tidak boleh di-commit ke Git** dan tidak boleh dibagikan, karena berisi rahasia. Laravel sudah memasukkannya ke `.gitignore`.
- Yang di-commit adalah `.env.example`, yaitu contoh isi `.env` **tanpa** data rahasia. Orang lain yang meng-clone project menyalinnya menjadi `.env` miliknya sendiri.
- Saat kamu membuat project dengan `laravel new`, file `.env` dan `APP_KEY`-nya sudah dibuatkan otomatis.

## Menyiapkan `.env` pada project hasil clone

```bash
cp .env.example .env
php artisan key:generate
```

Di Windows PowerShell, `cp` juga bisa dipakai. Alternatifnya `Copy-Item .env.example .env`.

Setelah itu isi `DB_*` dan pengaturan lain sesuai komputermu, lalu jalankan `php artisan migrate`.

## Bentuk isi file

Satu baris satu pengaturan, dengan format `NAMA=nilai`. Baris berawalan `#` adalah komentar. Kalau nilainya mengandung spasi, apit dengan tanda kutip:

```ini
APP_NAME="Toko Saya"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://my-app.test

# Database
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=toko
DB_USERNAME=root
DB_PASSWORD=
```

Nilai bawaan bisa berbeda di tiap versi Laravel (misalnya database default SQLite), jadi selalu cek isi `.env` project-mu sendiri.

## Variabel yang paling sering dipakai

| Variabel | Fungsi | Contoh nilai |
|---|---|---|
| `APP_NAME` | Nama aplikasi, dipakai di judul dan email | `"Toko Saya"` |
| `APP_ENV` | Jenis lingkungan: `local`, `testing`, `production` | `local` |
| `APP_KEY` | Kunci enkripsi untuk sesi dan data terenkripsi. Dibuat oleh `php artisan key:generate` | `base64:...` |
| `APP_DEBUG` | `true` menampilkan detail error lengkap. **Wajib `false` di production** | `true` |
| `APP_URL` | Alamat dasar aplikasi, dipakai untuk membuat link | `http://my-app.test` |
| `DB_CONNECTION` | Jenis database: `sqlite`, `mysql`, `mariadb`, `pgsql`, `sqlsrv` | `mysql` |
| `DB_HOST`, `DB_PORT` | Lokasi server database | `127.0.0.1`, `3306` |
| `DB_DATABASE` | Nama database (untuk SQLite berupa file) | `toko` |
| `DB_USERNAME`, `DB_PASSWORD` | Akun untuk masuk ke database | `root`, `rahasia` |
| `SESSION_DRIVER` | Tempat penyimpanan sesi login | `database` |
| `CACHE_STORE` | Tempat penyimpanan cache | `database` |
| `QUEUE_CONNECTION` | Tempat antrean job | `database` |
| `MAIL_MAILER` | Cara mengirim email. `log` hanya menulis email ke file log, aman untuk development | `log` |
| `FILESYSTEM_DISK` | Lokasi penyimpanan file upload | `local` |

## Membaca nilai `.env` di dalam kode

Ada aturan penting: **panggil `env()` hanya di file dalam folder `config/`**. Di bagian lain aplikasi, baca lewat `config()`.

Contoh menambah pengaturan sendiri, misalnya kunci API pembayaran:

**1. Tambahkan di `.env`:**
```ini
PAYMENT_API_KEY=abc123
```

**2. Daftarkan di `config/services.php`:**
```php
'payment' => [
    'key' => env('PAYMENT_API_KEY'),
],
```

**3. Pakai di kode (controller, service, dst.):**
```php
$key = config('services.payment.key');
```

!!! warning "Kenapa tidak `env()` langsung di controller?"
    Di production, `php artisan optimize` atau `config:cache` menyimpan semua konfigurasi ke dalam cache. Setelah itu `env()` yang dipanggil di luar folder `config/` akan mengembalikan `null`. Membaca lewat `config()` aman baik dengan maupun tanpa cache.

## Setelah mengubah `.env`

Perubahan di `.env` tidak selalu langsung terbaca. Lakukan sesuai kondisi:

| Kondisi | Yang perlu dilakukan |
|---|---|
| Sedang development biasa | Biasanya langsung terbaca saat halaman di-refresh |
| Perubahan tidak muncul (config sudah ter-cache) | `php artisan config:clear` |
| Queue worker sedang berjalan | Hentikan (Ctrl+C) lalu jalankan lagi, karena worker menyimpan konfigurasi lama di memori |
| Ingin memeriksa nilai yang sedang dipakai | `php artisan config:show database` |

## Praktik yang aman

- Jangan pernah meng-commit `.env`, mengirimnya lewat chat, atau menaruhnya di folder yang bisa diakses browser. Hanya folder `public/` yang boleh terbuka ke publik.
- Di production, pastikan `APP_ENV=production` dan `APP_DEBUG=false`. Kalau `APP_DEBUG=true`, pengunjung bisa melihat detail error, termasuk sebagian isi konfigurasi.
- Jangan mengganti `APP_KEY` pada aplikasi yang sudah berjalan, kecuali kamu paham akibatnya. Data terenkripsi dan sesi login yang sudah ada bisa jadi tidak terbaca lagi.
- Kalau ada kunci rahasia yang tidak sengaja ter-commit, anggap sudah bocor dan buat kunci baru dari penyedia layanannya.

## Lanjut belajar

- [Composer](composer.md): memasang package PHP baru.
- [npm dan npx](npm.md): package JavaScript dan deploy aset.
- Praktik langsung: [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md).
