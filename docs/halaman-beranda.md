# Halaman Beranda (Dashboard)

Panduan ini menjelaskan cara membuat halaman beranda (dashboard) sederhana untuk aplikasi Laravel Anda. Halaman ini dapat menampilkan pesan selamat datang, statistik singkat (misalnya jumlah buku), dan tautan ke berbagai fitur aplikasi seperti CRUD buku.

!!! note "Prasyarat"
    Anda sudah memiliki aplikasi Laravel yang berjalan (misalnya dari tutorial [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md)). Jika belum, silakan ikuti terlebih dahulu tutorial tersebut untuk membuat proyek Laravel dasar.

---

## 1. Membuat Controller

Buat controller bernama `HomeController` dengan method `index`:

```bash
php artisan make:controller HomeController
```

Buka file `app/Http/Controllers/HomeController.php` dan ubah menjadi:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Book; // Asumsi Anda sudah memiliki model Book dari tutorial CRUD

class HomeController extends Controller
{
    /**
     * Tampilkan halaman beranda.
     *
     * @return \Illuminate\View\View
     */
    public function index()
    {
        // Ambil statistik sederhana (opsional)
        $totalBooks = Book::count();
        $recentBooks = Book::latest()->take(5)->get();

        // Kirim data ke view
        return view('home', [
            'totalBooks' => $totalBooks,
            'recentBooks' => $recentBooks,
        ]);
    }
}
```

Jika Anda tidak ingin menampilkan statistik, cukup kembalikan view tanpa data:

```php
public function index()
{
    return view('home');
}
```

---

## 2. Membuat View

Buat file Blade bernama `home.blade.php` di dalam folder `resources/views`. Isi file tersebut dengan konten berikut:

```blade
@extends('layouts.app')

@section('title', 'Beranda')

@section('content')
<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">
                    <h3 class="text-center">Selamat Datang di Aplikasi Laravel Anda</h3>
                </div>

                <div class="card-body">
                    <p>
                        Ini adalah halaman beranda dari aplikasi Laravel yang Anda bangun.
                        Dari sini Anda dapat mengakses berbagai fitur yang telah Anda pelajari.
                    </p>

                    <!-- Statistik Sederhana -->
                    <div class="row mb-4">
                        <div class="col-md-6">
                            <div class="bg-light p-3 rounded">
                                <h5 class="mb-0">Total Buku</h5>
                                <p class="fs-4 mb-0">{{ $totalBooks }}</p>
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="bg-light p-3 rounded">
                                <h5 class="mb-0">Buku Baru</h5>
                                <p class="fs-4 mb-0">{{ $recentBooks->count() }} buku</p>
                            </div>
                        </div>
                    </div>

                    <!-- Tautan ke Fitur Utama -->
                    <div class="d-grid gap-2 d-md-flex justify-content-md-start">
                        <a href="{{ route('book.index') }}" class="btn btn-primary me-md-2">
                            📚 Daftar Buku
                        </a>
                        <a href="{{ route('book.create') }}" class="btn btn-success me-md-2">
                            ➕ Tambah Buku
                        </a>
                        @if (Route::has('category.index'))
                            <a href="{{ route('category.index') }}" class="btn btn-info me-md-2">
                                🏷️ Kelola Kategori
                            </a>
                        @endif
                        @if (Route::has('author.index'))
                            <a href="{{ route('author.index') }}" class="btn btn-outline-secondary">
                                � Penulis
                            </a>
                        @endif
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

Catatan: View ini mengasumsikan Anda menggunakan layout `resources/views/layouts/app.blade.php` yang sudah ada dari tutorial CRUD. Jika belum, Anda dapat membuat layout sederhana atau mengubahnya sesuai kebutuhan.

---

## 3. Menambahkan Route

Buka file `routes/web.php` dan tambahkan route untuk halaman beranda:

```php
use App\Http\Controllers\HomeController;

// Halaman beranda
Route::get('/', [HomeController::class, 'index'])->name('home');

// Pastikan resource route untuk buku masih ada
use App\Http\Controllers\BookController;
Route::resource('book', BookController::class);

// Jika Anda memiliki route untuk kategori atau penulis, pastikan juga terdaftar
// Route::resource('category', CategoryController::class);
// Route::resource('author', AuthorController::class);
```

Route di atas menggunakan nama `home` sehingga Anda dapat memanggilnya di mana saja dengan `route('home')`.

---

## 4. Menggunakan di Navigasi Menu (Opsional)

Jika Anda sudah membuat modul navigasi menu ([Komponen Blade: Navigasi Menu](komponen-navigasi.md)), Anda dapat mengubah item menu Beranda menjadi menggunakan nama route `home`:

```blade
<!-- Dalam komponen nav-menu atau view composer -->
['label' => 'Beranda', 'route' => 'home'],
```

Cara ini lebih baik daripada menggunakan URL langsung (`'url' => '/'`) karena:
- Tidak tergantung pada struktur URL yang mungkin berubah.
- Memungkinkan Anda dengan mudah mengubah destinasi route di satu tempat saja (di file `routes/web.php`).
- Selalu konsisten dengan nama route yang didefinisikan.

Jika Anda belum membuat controller dan route untuk `home`, maka menggunakan `'route' => 'home'` akan menyebabkan error. Pastikan langkah 1–3 di atas sudah diselesaikan sebelum menggunakan nama route.

---

## 5. Ringkasan Perintah yang Digunakan

| Command                                                     | Fungsi                                   |
| ----------------------------------------------------------- | ---------------------------------------- |
| `php artisan make:controller HomeController`                | Membuat controller `HomeController`      |
| `touch resources/views/home.blade.php`                      | Membuat file view kosong (bisa juga pakai `php artisan make:view` jika tersedia) |
| `php artisan serve`                                         | Menjalankan server pengembangan          |

---

## 6. Tantangan Lanjutan

Setelah halaman beranda dasar berjalan, Anda bisa mencoba:
- Tambahkan navigasi menu menggunakan komponen Blade yang dapat digunakan kembali: [Komponen Blade: Navigasi Menu](komponen-navigasi.md)
- Menambahkan grafik atau chart menggunakan Chart.js atau library lain untuk menampilkan statistik buku secara visual.
- Menampilkan notifikasi atau flash message ketika ada perubahan data (misalnya setelah menambah buku).
- Mengimplementasikan autentikasi sehingga halaman beranda hanya dapat diakses oleh pengguna yang sudah login (gunakan Laravel Breeze atau Jetstream).
- Menambahkan widget seperti kalender, tarek, atau daftar tugas.
- Menggunakan view composer untuk menyediakan data seperti total buku ke semua view secara otomatis.

---

Materi terkait: [Controllers](https://laravel.com/controllers) · [Views](https://laravel.com/views) · [Routing](https://laravel.com/routing) · [Eloquent ORM](https://laravel.com/eloquent) · [CRUD Data Buku](aplikasi-ujicoba-crud.md)