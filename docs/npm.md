# npm dan npx

Laravel memakai **Vite** untuk mengolah file CSS dan JavaScript (dan Tailwind, Vue, React, dst.). Alat pendukungnya berasal dari ekosistem Node.js:

- **Node.js**: menjalankan JavaScript di komputer kamu.
- **npm**: pengelola paket JavaScript, terpasang bersama Node.js. Fungsinya mirip Composer untuk PHP.
- **npx**: menjalankan program dari sebuah package tanpa perlu memasangnya secara global.

Cek keberadaannya dengan `node --version` dan `npm --version`. Gunakan Node.js versi LTS terbaru. Herd sudah membawa Node.js dan nvm; di Linux, pasang lewat nvm (lihat [panduan Linux](linux.md)).

## Berkas yang perlu dikenal

| Berkas atau folder | Fungsi |
|---|---|
| `package.json` | Daftar package JavaScript dan `scripts` (jalan pintas command). **Di-commit.** |
| `package-lock.json` | Catatan versi persis yang terpasang. **Di-commit.** |
| `node_modules/` | Tempat package diunduh. **Tidak di-commit.** Bisa dibuat ulang dengan `npm install`. |
| `vite.config.js` | Pengaturan Vite dan plugin-nya. |
| `resources/js/app.js` dan `resources/css/app.css` | File sumber yang kamu edit. |
| `public/build/` | Hasil `npm run build`. Dibuat otomatis, jangan diedit. |

## Command npm

| Command | Penjelasan | Contoh |
|---|---|---|
| `npm install` | Memasang semua package di `package.json`. Dipakai setelah clone project. | `npm install` |
| `npm install nama` | **Menambah package baru** ke `dependencies`. | `npm install chart.js` |
| `npm install -D nama` | Menambah package yang hanya dibutuhkan saat development atau build (plugin Vite, dst.). Bentuk panjangnya `--save-dev`. | `npm install -D @vitejs/plugin-vue` |
| `npm uninstall nama` | Menghapus package. | `npm uninstall chart.js` |
| `npm ci` | Memasang package **persis sesuai `package-lock.json`**, dengan menghapus `node_modules` lebih dulu. Lebih cepat dan konsisten, cocok untuk deploy. | `npm ci` |
| `npm run dev` | Menjalankan **server Vite untuk development**. Perubahan kode langsung tampil di browser tanpa refresh (HMR). | `npm run dev` |
| `npm run build` | Membuat **file production** yang sudah dikompres di `public/build`. | `npm run build` |
| `npm run` | Menampilkan semua script yang tersedia di `package.json`. | `npm run` |
| `npm outdated` | Menampilkan package yang punya versi lebih baru. | `npm outdated` |
| `npm update` | Memperbarui package dalam batas versi di `package.json`. | `npm update` |
| `npm audit` | Memeriksa celah keamanan yang diketahui pada package. | `npm audit` |

## `npm run dev` untuk development

```bash
npm run dev
```

Biarkan terminal ini terus berjalan selama kamu mengerjakan tampilan. Hentikan dengan **Ctrl+C**.

Kamu juga bisa memakai `composer run dev`, yang menjalankan Vite bersama server dan queue dalam satu command.

Halaman Blade harus memuat file lewat `@vite`:
```blade
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

!!! tip "Muncul error `Vite manifest not found`?"
    Artinya belum ada hasil build. Jalankan `npm run dev` (untuk development) atau `npm run build`.

## Memasang library atau plugin baru

Contoh: menambah library grafik **Chart.js**.

**1. Pasang**
```bash
npm install chart.js
```

**2. Impor di `resources/js/app.js`**
```javascript
import Chart from 'chart.js/auto';

window.Chart = Chart;
```

**3. Jalankan Vite dan pakai di Blade**
```bash
npm run dev
```

Untuk **plugin Vite** (misalnya untuk Vue), pasang dengan `-D`, lalu daftarkan di `vite.config.js` sesuai dokumentasi plugin tersebut:
```bash
npm install -D @vitejs/plugin-vue
```

## `npx`: menjalankan tanpa memasang global

`npx` menjalankan program dari package. Kalau package sudah ada di `node_modules`, itu yang dipakai. Kalau belum, ia mengunduhnya sementara.

| Contoh | Fungsi |
|---|---|
| `npx vite build` | Menjalankan build Vite langsung (hasilnya sama dengan `npm run build`). |
| `npx vite --version` | Melihat versi Vite yang terpasang di project. |
| `npx npm-check-updates` | Melihat daftar package yang bisa dinaikkan versinya, termasuk lompatan versi mayor. |

Pola umumnya: **`npm run`** untuk script yang sudah ada di `package.json`, **`npx`** untuk menjalankan program sekali pakai.

## Deploy: menyiapkan aset production

Di production **jangan** menjalankan `npm run dev`. Yang dipakai adalah hasil `npm run build`. Urutan umum di server, setelah kode terbaru diambil (`git pull`):

```bash
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan optimize
```

| Langkah | Alasan |
|---|---|
| `composer install --no-dev -o` | Memasang package PHP tanpa alat development |
| `npm ci` | Memasang package JavaScript persis sesuai lock file |
| `npm run build` | Membuat file CSS dan JS production di `public/build` |
| `php artisan migrate --force` | Menjalankan migrasi. `--force` diperlukan agar tidak berhenti menanyakan konfirmasi di production |
| `php artisan optimize` | Menyimpan cache config, route, dan view |

Jika server tidak punya Node.js, jalankan `npm run build` di komputer kamu atau di CI, lalu unggah folder `public/build` ke server. Pastikan juga `.env` di server sudah diatur (lihat [File `.env`](env.md)), dan `php artisan storage:link` dijalankan sekali kalau memakai file upload.

## Masalah yang sering muncul

| Masalah | Solusi |
|---|---|
| Perubahan CSS atau JS tidak tampil | Pastikan `npm run dev` sedang berjalan, atau jalankan `npm run build` ulang. Coba refresh keras (Ctrl+Shift+R). |
| `Vite manifest not found` | Jalankan `npm run dev` atau `npm run build`. |
| `command not found: npm` atau `node` | Node.js belum terpasang atau terminal belum dibuka ulang. Cek `node --version`. |
| Error saat `npm install` atau `node_modules` rusak | Hapus `node_modules` lalu pasang ulang. Bash: `rm -rf node_modules && npm install`. PowerShell: `Remove-Item -Recurse -Force node_modules; npm install`. |
| Error konflik versi (`ERESOLVE`) | Baca pesannya dulu, biasanya ada package yang belum cocok dengan versi lain. Menambahkan `--legacy-peer-deps` bisa melewatinya, tetapi gunakan sebagai jalan terakhir. |

## Lanjut belajar

- [File `.env`](env.md): pengaturan aplikasi dan rahasia.
- [Composer](composer.md): memasang package PHP baru.
- Praktik langsung: [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md).
