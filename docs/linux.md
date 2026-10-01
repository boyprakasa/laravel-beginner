# Panduan Laravel 13 dengan Valet di Linux

Panduan instalasi Laravel dari nol di Linux memakai Valet, lengkap dengan command Valet dan Artisan beserta penjelasannya untuk pemula. Perintah dasar tidak diubah; shortcut hanya tambahan.

!!! warning "Valet di Linux bukan versi resmi"
    Laravel Valet resmi hanya untuk macOS. Di Linux dipakai **Valet Linux+**, port buatan komunitas yang tidak dikelola tim Laravel. Fitur dan kecepatan update-nya bisa berbeda dari Valet di macOS maupun Herd. Panduan ini untuk distro berbasis **Ubuntu/Debian** (Ubuntu, Linux Mint, Zorin, Debian). Fedora dan Arch disebut didukung oleh proyeknya, tetapi paket dan langkahnya berbeda dan tidak dibahas di sini.

Laravel 13 membutuhkan **PHP 8.3 atau lebih baru**. Berbeda dengan Herd, PHP, Composer, dan Node.js **tidak** terpasang otomatis, jadi kita pasang sendiri.

## 1. Instalasi

### 1.1 Persiapan sistem
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y software-properties-common curl git unzip zip
```

### 1.2 Tambah PPA Ondřej Surý (PHP dan Nginx)
Repository bawaan Ubuntu sering hanya berisi PHP versi lama. PPA ini menyediakan versi PHP terbaru:
```bash
sudo add-apt-repository ppa:ondrej/php
sudo add-apt-repository ppa:ondrej/nginx
sudo apt update
```
PPA ini untuk Ubuntu dan turunannya (termasuk Linux Mint dan Zorin). Di Debian murni, gunakan repository dari deb.sury.org.

### 1.3 Install PHP
Contoh memakai PHP 8.4 (boleh 8.3, ganti angkanya):
```bash
sudo apt install -y php8.4-cli php8.4-fpm php8.4-common php8.4-mbstring php8.4-xml \
  php8.4-curl php8.4-zip php8.4-bcmath php8.4-intl php8.4-gd php8.4-sqlite3 php8.4-mysql
php --version        # harus 8.3 atau lebih baru
```
Kalau ada beberapa versi PHP terpasang dan `php --version` menampilkan yang lama, pilih yang benar:
```bash
sudo update-alternatives --set php /usr/bin/php8.4
```

### 1.4 Install Composer
```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

### 1.5 Install Node.js
Versi Node.js dari `apt` sering terlalu lama untuk Vite. Disarankan memakai **nvm** (petunjuk di [github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)), lalu:
```bash
nvm install --lts
node --version
```

### 1.6 Paket pendukung Valet
```bash
sudo apt install -y libnss3-tools jq xsel
```
`libnss3-tools` dipakai agar sertifikat HTTPS lokal bisa dipercaya browser. Kalau `valet install` nanti meminta paket tambahan, ikuti pesan errornya.

### 1.7 Pastikan command Composer global bisa dipanggil
Cek lokasi folder bin Composer:
```bash
composer global config bin-dir --absolute
```
Biasanya hasilnya `/home/namakamu/.config/composer/vendor/bin`. Tambahkan ke PATH:
```bash
echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.bashrc
source ~/.bashrc
```
Kalau hasil perintah di atas berbeda (misalnya `~/.composer/vendor/bin`), sesuaikan path-nya. Kalau memakai zsh, ganti `~/.bashrc` dengan `~/.zshrc`.

### 1.8 Install installer Laravel
```bash
composer global require laravel/installer
laravel --version
```

### 1.9 Install Valet Linux+
```bash
composer global require ahmdadl/valet-linux-plus
valet install
```
`valet install` meminta password `sudo` dan menyiapkan layanan yang dibutuhkan (web server, DNS lokal, dan database). Opsi tambahan:
```bash
valet install --with-pgsql     # sekaligus menyiapkan PostgreSQL
valet install --mariadb        # pakai MariaDB, bukan MySQL
```
Cek hasilnya:
```bash
valet status
```

### 1.10 Siapkan folder project (park)
```bash
mkdir -p ~/Sites
cd ~/Sites
valet park
```
Project apa pun di `~/Sites` otomatis dilayani sebagai `namafolder.test`. Domain akhiran default bisa dicek dengan `valet tld`.

### 1.11 Buat project
```bash
cd ~/Sites
laravel new my-app
```
Installer menanyakan beberapa pilihan:

- **Starter kit**: pilih *None* untuk mulai dari nol, atau React, Vue, Livewire kalau butuh login dan dashboard bawaan.
- **Testing framework**: Pest atau PHPUnit.
- **Database**: SQLite (default, tanpa server), MySQL, MariaDB, PostgreSQL, atau SQL Server.

Pertanyaan bisa dilewati dengan flag:
```bash
laravel new my-app --pest --database=mysql --git --npm
laravel new my-app --no-interaction
```

### 1.12 Jalankan dan buka
```bash
cd my-app
npm install && npm run build
composer run dev                # server + queue + log + Vite sekaligus
valet open                      # atau buka http://my-app.test di browser
```

### 1.13 Database dan HTTPS
```bash
php artisan migrate
valet secure                    # https://my-app.test
```
Firefox dan Chrome mengelola sertifikat di Linux dengan cara berbeda, jadi setelah `valet secure` kamu mungkin perlu menutup dan membuka ulang browser agar sertifikatnya dipercaya.

## 2. Command Valet

Daftar di bawah berisi command yang umum dipakai. Untuk daftar lengkap versi Linux+, jalankan `valet list`, dan untuk penjelasan satu command jalankan `valet help namacommand`.

| Command | Fungsi | Contoh |
|---|---|---|
| `valet install` | Memasang dan mengonfigurasi Valet beserta layanan pendukungnya. Jalankan lagi setelah update atau kalau ada yang rusak. | `valet install` |
| `valet status` | Memeriksa apakah layanan Valet berjalan. | `valet status` |
| `valet start` / `valet stop` | Menyalakan atau mematikan layanan Valet. | `valet stop` |
| `valet restart` | Merestart layanan kalau site tiba-tiba tidak bisa dibuka. | `valet restart` |
| `valet park` | Mendaftarkan folder saat ini sebagai folder parked, sehingga semua project di dalamnya otomatis punya domain `.test`. | `valet park` |
| `valet forget` | Menghapus folder dari daftar parked. | `valet forget` |
| `valet link` | Melayani satu folder dengan nama domain pilihan, cocok untuk project di luar folder parked. | `valet link toko` lalu buka `toko.test` |
| `valet unlink` | Menghapus link yang dibuat `valet link`. | `valet unlink toko` |
| `valet secure` / `valet unsecure` | Mengaktifkan atau mematikan HTTPS lokal untuk site. | `valet secure` |
| `valet open` | Membuka site dari folder saat ini di browser. | `valet open` |
| `valet use` | Mengganti versi PHP global. | `valet use 8.3` |
| `valet isolate` / `valet unisolate` | Mengunci satu site ke versi PHP tertentu, atau melepasnya. Format versi bisa berbeda, jadi cek `valet help isolate`. | `valet isolate 8.4` |
| `valet isolated` | Menampilkan daftar site yang sudah di-isolate. | `valet isolated` |
| `valet which-php` | Menampilkan PHP mana yang dipakai site saat ini. | `valet which-php` |
| `valet php` / `valet composer` | Menjalankan PHP atau Composer dengan versi yang dipakai site saat ini. | `valet composer install` |
| `valet uninstall` | Menghapus konfigurasi Valet dari sistem. | `valet uninstall` |

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

!!! warning "Perhatian"
    `migrate:fresh` dan `migrate:refresh` **menghapus data**. Pakai hanya di development, jangan di production.

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

### 4.1 Alias `a` untuk `php artisan` (bash)

Edit `~/.bashrc`:
```bash
nano ~/.bashrc
```
Tambahkan:
```bash
alias a='php artisan'
```
Simpan (Ctrl+O, Enter, Ctrl+X), lalu muat ulang:
```bash
source ~/.bashrc
```
Kalau memakai zsh, lakukan hal yang sama pada `~/.zshrc`. Contoh pemakaian: `a migrate`, `a make:model Post -a`.

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

## 5. Kalau Valet bermasalah

- **Site tidak terbuka:** jalankan `valet restart`, lalu cek layanan PHP-FPM dengan `systemctl status php8.4-fpm` (sesuaikan versi PHP-nya).
- **Setelah mengganti versi PHP:** jalankan `valet install` lagi agar konfigurasi menyesuaikan.
- **Command `valet` tidak ditemukan:** PATH Composer belum benar. Ulangi langkah 1.7 lalu buka terminal baru.

Valet di Linux adalah proyek komunitas, jadi kadang ada masalah yang tidak terjadi di macOS. Sebagai alternatif tanpa Valet, kamu bisa memasang PHP, Composer, dan installer Laravel sekaligus dengan skrip resmi php.new:
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```
Setelah itu buat project dengan `laravel new my-app`, lalu jalankan `composer run dev` dan buka `http://localhost:8000`.

## 6. Langkah berikutnya

- Buat dan jelajahi halaman beranda/dashboard: [Halaman Beranda](halaman-beranda.md)
- Pelajari dasar-dasarnya: [File `.env`](env.md), [Composer](composer.md), dan [npm dan npx](npm.md).
- Praktik langsung: [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md).
