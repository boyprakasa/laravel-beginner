---
layout: default
title: Panduan Laravel 13 dengan Herd di Windows
---

# Panduan Laravel 13 dengan Herd di Windows

Panduan instalasi Laravel dari nol memakai Laravel Herd di Windows, lengkap dengan daftar command Herd dan Artisan beserta penjelasannya untuk pemula. Perintah dasar tidak diubah; shortcut hanya tambahan.

Laravel 13 membutuhkan **PHP 8.3 atau lebih baru**. Herd sudah membawa PHP terbaru, jadi kamu tidak perlu install PHP sendiri.

## 1. Instalasi

### 1.1 Install Herd
1. Unduh installer Herd untuk Windows dari [herd.laravel.com](https://herd.laravel.com).
2. Jalankan installer sampai selesai, lalu buka Herd sekali.
3. Herd otomatis memasang PHP, Composer, Node.js, npm, dan installer `laravel`.

### 1.2 Verifikasi
Buka **PowerShell** atau **Windows Terminal** yang baru (terminal lama belum mengenali PATH yang baru):

```powershell
herd --version
php --version        # harus 8.3 atau lebih baru
composer --version
laravel --version
node --version
```

Kalau `laravel --version` menampilkan versi lama, update aplikasi Herd. Installer `laravel` yang dipakai berasal dari Herd, bukan dari `composer global`.

### 1.3 Masuk ke folder parked
```powershell
cd $HOME\Herd
```
Folder `C:\Users\NamaKamu\Herd` adalah folder *parked*. Project apa pun di dalamnya otomatis dilayani Herd lewat alamat `namafolder.test`.

### 1.4 Buat project
```powershell
laravel new my-app
```
Installer menanyakan beberapa pilihan:
- **Starter kit**: pilih *None* untuk mulai dari nol, atau React, Vue, Livewire kalau butuh login dan dashboard bawaan.
- **Testing framework**: Pest atau PHPUnit.
- **Database**: SQLite (default, tanpa server), MySQL, MariaDB, PostgreSQL, atau SQL Server.

Pertanyaan bisa dilewati dengan flag:
```powershell
laravel new my-app --pest --database=mysql --git --npm
laravel new my-app --no-interaction
```

### 1.5 Jalankan dan buka
```powershell
cd my-app
herd open                    # membuka http://my-app.test
npm install; npm run build   # di PowerShell 5.1 pakai titik koma, bukan &&
composer run dev             # server + queue + log + Vite sekaligus
```

### 1.6 Database dan HTTPS
```powershell
php artisan migrate
herd secure                  # https://my-app.test
```

## 2. Command Herd

| Command | Fungsi | Contoh |
|---|---|---|
| `herd open` | Membuka site dari folder saat ini di browser. Herd mencocokkan nama folder dengan domain `.test`. | `herd open` |
| `herd secure` | Membuat sertifikat SSL lokal supaya site bisa diakses lewat `https://`. Berguna kalau fitur seperti OAuth atau cookie aman butuh HTTPS. | `herd secure` |
| `herd unsecure` | Mengembalikan site ke `http://`. | `herd unsecure` |
| `herd park` | Mendaftarkan folder sebagai folder parked, sehingga semua project di dalamnya otomatis punya domain `.test`. Jalankan di dalam folder yang dimaksud. | `herd park` |
| `herd forget` | Menghapus folder dari daftar parked. | `herd forget` |
| `herd link` | Melayani satu folder dengan nama domain pilihan, cocok untuk project di luar folder parked. | `herd link toko` lalu buka `toko.test` |
| `herd unlink` | Menghapus link yang dibuat `herd link`. | `herd unlink toko` |
| `herd isolate` | Mengunci satu project ke versi PHP tertentu, sementara project lain tetap memakai versi global. | `herd isolate 8.4` |
| `herd use` | Mengganti versi PHP global untuk semua project yang tidak di-isolate. | `herd use 8.3` |
| `herd php` | Menjalankan PHP dengan versi yang dipakai project saat ini. | `herd php -v` |
| `herd composer` | Menjalankan Composer dengan PHP yang dipakai project saat ini. | `herd composer install` |
| `herd restart` | Merestart layanan Herd kalau site tiba-tiba tidak bisa dibuka. | `herd restart` |

## 3. Command Artisan

Artisan adalah alat command line bawaan Laravel. Semuanya dijalankan dari dalam folder project dengan awalan `php artisan`.

### 3.1 Info dan bantuan

| Command | Penjelasan | Contoh |
|---|---|---|
| `list` | Menampilkan semua command yang tersedia. Titik awal yang bagus kalau lupa nama command. | `php artisan list` |
| `help` | Menampilkan penjelasan lengkap satu command, termasuk semua opsi (flag) yang bisa dipakai. | `php artisan help make:model` |
| `about` | Ringkasan project: versi Laravel, versi PHP, driver cache, database, dan status environment. Berguna saat debugging atau minta bantuan orang lain. | `php artisan about` |
| `env` | Menampilkan environment yang sedang aktif (`local`, `production`, dst.). | `php artisan env` |
| `tinker` | Console interaktif untuk mencoba kode PHP dengan konteks aplikasi kamu, tanpa harus membuat halaman atau route. Ketik `exit` untuk keluar. | `php artisan tinker` lalu `User::count()` |

### 3.2 Generator (`make:*`)

Command `make:*` membuat file kerangka (boilerplate) di folder yang benar, jadi kamu tidak perlu membuat file dan menulis namespace-nya manual.

| Command | Penjelasan | Contoh |
|---|---|---|
| `make:model` | Membuat *model*, yaitu class yang mewakili satu tabel database (misalnya `Product` untuk tabel `products`). Bisa sekaligus membuat file pendukung lewat flag (lihat 3.3). | `php artisan make:model Product -mfsc` |
| `make:controller` | Membuat *controller*, tempat logika yang menerima request dan mengembalikan response. Flag `--resource` menyiapkan method standar (index, store, show, update, destroy). | `php artisan make:controller ProductController --resource` |
| `make:migration` | Membuat *migration*, yaitu file yang menuliskan perubahan struktur database (buat tabel, tambah kolom) dalam bentuk kode. Ini seperti "riwayat versi" skema database. | `php artisan make:migration create_orders_table` |
| `make:seeder` | Membuat *seeder*, yaitu class untuk mengisi data awal ke database (misalnya akun admin). | `php artisan make:seeder AdminSeeder` |
| `make:factory` | Membuat *factory*, yaitu pembuat data palsu (dummy) untuk testing atau development. | `php artisan make:factory ProductFactory` |
| `make:request` | Membuat *form request*, yaitu class khusus untuk aturan validasi input, supaya controller tetap ringkas. | `php artisan make:request StoreProductRequest` |
| `make:middleware` | Membuat *middleware*, yaitu "penjaga gerbang" yang memeriksa request sebelum sampai ke controller (misalnya cek apakah user adalah admin). | `php artisan make:middleware CekAdmin` |
| `make:policy` | Membuat *policy*, yaitu aturan siapa yang boleh melakukan apa terhadap suatu model (misalnya hanya pemilik yang boleh edit). | `php artisan make:policy ProductPolicy --model=Product` |
| `make:resource` | Membuat *API resource*, yang mengatur bentuk JSON yang dikirim ke pengguna API. | `php artisan make:resource ProductResource` |
| `make:job` | Membuat *job*, yaitu tugas berat yang dijalankan di latar belakang lewat queue (misalnya mengirim email massal). | `php artisan make:job KirimEmailNota` |
| `make:event` dan `make:listener` | Event menandai sesuatu telah terjadi (pesanan dibuat), listener adalah reaksi terhadapnya (kirim notifikasi). | `php artisan make:event OrderDibuat` |
| `make:mail` | Membuat class email beserta template-nya. | `php artisan make:mail InvoiceMail` |
| `make:notification` | Membuat notifikasi yang bisa dikirim lewat email, database, atau kanal lain. | `php artisan make:notification PesananSelesai` |
| `make:command` | Membuat command Artisan buatan sendiri, misalnya untuk tugas rutin. | `php artisan make:command HitungStok` |
| `make:test` | Membuat file test otomatis. Tambah `--pest` untuk gaya Pest. | `php artisan make:test ProductTest --pest` |
| `make:component` | Membuat *Blade component*, potongan tampilan yang bisa dipakai ulang. | `php artisan make:component Alert` |
| `make:enum` | Membuat enum, yaitu daftar nilai tetap (misalnya status: pending, paid, cancelled). | `php artisan make:enum OrderStatus` |

### 3.3 Flag singkat pada `make:model`

| Flag | Artinya | Hasil tambahan |
|---|---|---|
| `-m` | migration | file migrasi tabelnya |
| `-f` | factory | data dummy |
| `-s` | seeder | pengisi data awal |
| `-c` | controller | controller kosong |
| `-r` | resource | controller dengan method lengkap |
| `-a` | all | semua file pendukung sekaligus |

Contoh: `php artisan make:model Product -mfs` membuat model, migrasi, factory, dan seeder dalam satu perintah.

### 3.4 Database

| Command | Penjelasan | Contoh |
|---|---|---|
| `migrate` | Menjalankan semua migrasi yang belum pernah dijalankan, sehingga tabel dibuat atau diubah. | `php artisan migrate` |
| `migrate:status` | Melihat migrasi mana yang sudah dan belum dijalankan. | `php artisan migrate:status` |
| `migrate:rollback` | Membatalkan migrasi batch terakhir. `--step=1` membatalkan satu langkah saja. | `php artisan migrate:rollback --step=1` |
| `migrate:fresh` | **Menghapus semua tabel**, lalu menjalankan semua migrasi dari awal. Tambah `--seed` untuk sekaligus mengisi data awal. | `php artisan migrate:fresh --seed` |
| `migrate:refresh` | Membatalkan semua migrasi lalu menjalankannya lagi. Hasil akhirnya mirip `fresh`, tetapi lewat proses rollback. | `php artisan migrate:refresh` |
| `db:seed` | Menjalankan seeder untuk mengisi data awal. | `php artisan db:seed --class=AdminSeeder` |
| `db:show` dan `db:table` | Melihat ringkasan database atau struktur satu tabel, tanpa aplikasi database terpisah. | `php artisan db:table users` |
| `model:show` | Menampilkan atribut dan relasi sebuah model. | `php artisan model:show Product` |

> **Perhatian:** `migrate:fresh` dan `migrate:refresh` **menghapus data**. Pakai hanya di development, jangan di production.

### 3.5 Route, cache, dan optimasi

| Command | Penjelasan | Contoh |
|---|---|---|
| `route:list` | Menampilkan semua route (URL, method, nama, controller). Pakai `--path` untuk memfilter. | `php artisan route:list --path=api` |
| `optimize` | Menyimpan cache config, route, event, dan view supaya aplikasi lebih cepat. Umumnya dipakai di production. | `php artisan optimize` |
| `optimize:clear` | Menghapus semua cache di atas. Jalankan saat perubahan kamu tidak muncul. | `php artisan optimize:clear` |
| `config:clear` | Menghapus cache konfigurasi. Berguna setelah mengubah `.env` dan hasilnya tidak berubah. | `php artisan config:clear` |
| `route:clear` | Menghapus cache route. | `php artisan route:clear` |
| `view:clear` | Menghapus cache tampilan Blade. | `php artisan view:clear` |
| `cache:clear` | Menghapus data cache aplikasi (bukan cache config atau route). | `php artisan cache:clear` |

### 3.6 Queue, jadwal, dan lainnya

| Command | Penjelasan | Contoh |
|---|---|---|
| `queue:work` | Menjalankan *worker* yang mengambil job dari antrean dan memprosesnya di latar belakang. `--tries=3` artinya dicoba maksimal 3 kali. | `php artisan queue:work --tries=3` |
| `queue:listen` | Mirip `queue:work`, tetapi otomatis memuat ulang kode. Nyaman untuk development, tapi lebih lambat. | `php artisan queue:listen` |
| `queue:failed` dan `queue:retry` | Melihat job yang gagal, lalu mengulanginya. | `php artisan queue:retry all` |
| `schedule:list` | Menampilkan tugas terjadwal beserta jadwal berikutnya. | `php artisan schedule:list` |
| `schedule:work` | Menjalankan scheduler di lokal, sehingga tidak perlu cron. | `php artisan schedule:work` |
| `key:generate` | Membuat `APP_KEY` di `.env`, kunci untuk enkripsi dan sesi. Wajib ada agar aplikasi berjalan. | `php artisan key:generate` |
| `storage:link` | Membuat symlink `public/storage` ke `storage/app/public`, agar file upload bisa diakses dari browser. | `php artisan storage:link` |
| `vendor:publish` | Menyalin file config atau asset dari sebuah package ke project kamu supaya bisa diedit. | `php artisan vendor:publish` |
| `install:api` | Menyiapkan fitur API (Sanctum, route `api.php`). | `php artisan install:api` |
| `install:broadcasting` | Menyiapkan fitur real-time (Reverb dan channel). | `php artisan install:broadcasting` |
| `down` dan `up` | Mengaktifkan atau mematikan mode maintenance. `--secret` memberi kamu jalur akses saat maintenance. | `php artisan down --secret=rahasia` |
| `test` | Menjalankan semua test dengan tampilan hasil yang rapi. `--filter` menjalankan sebagian saja. | `php artisan test --filter=ProductTest` |

## 4. Shortcut

Shortcut hanya tambahan. `php artisan` tetap bisa dipakai seperti biasa.

### 4.1 Alias `a` untuk `php artisan` (PowerShell)

Buat file profile kalau belum ada, lalu buka:
```powershell
if (!(Test-Path $PROFILE)) { New-Item -Type File -Path $PROFILE -Force }
notepad $PROFILE
```
Tambahkan:
```powershell
function a { php artisan @args }
```
Simpan, lalu buka PowerShell baru. Kalau muncul error soal *execution policy*, jalankan sekali:
```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```
Contoh pemakaian: `a migrate`, `a make:model Post -a`.

### 4.2 Shortcut lain

| Shortcut | Setara dengan | Fungsi |
|---|---|---|
| `composer setup` | install dependency, salin `.env`, `key:generate`, `migrate`, build asset | Menyiapkan project yang baru di-clone dalam satu langkah |
| `composer dev` | server + queue + log + Vite | Menjalankan semua kebutuhan development sekaligus |
| `composer test` | menjalankan test | Cara singkat menjalankan test |
| `make:model X -a` | model + migrasi + factory + seeder + policy + controller + request | Membuat semua file pendukung sekaligus |
| `migrate:fresh --seed` | reset semua tabel + isi data awal | Mengulang database dari nol saat development |
| `optimize:clear` | `config:clear`, `route:clear`, `view:clear`, dst. | Membersihkan semua cache sekaligus |

Nama script Composer di atas bisa dicek di bagian `scripts` pada `composer.json` project.
