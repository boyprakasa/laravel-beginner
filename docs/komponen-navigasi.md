# Komponen Blade: Navigasi Menu

Panduan ini menjelaskan cara membuat komponen Blade yang dapat digunakan kembali untuk navigasi menu (navbar) dalam aplikasi Laravel. Komponen ini akan menautkan ke berbagai route, menandai route yang aktif secara otomatis, dan dapat dengan mudah diperluas untuk menu dropdown atau menu samping.

!!! note "Prasyarat"
    Anda sudah memiliki aplikasi Laravel yang berjalan (misalnya dari tutorial [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md)). Jika belum, silakan ikuti terlebih dahulu tutorial tersebut untuk membuat proyek Laravel dasar.

---

## 1. Membuat Komponen Blade

Buat komponen Blade bernama `nav-menu` menggunakan perintah Artisan:

```bash
php artisan make:component NavMenu
```

Perintah ini akan membuat dua file:
- `app/View/Components/NavMenu.php` (kelas komponen)
- `resources/views/components/nav-menu.blade.php` (tampilan)

---

## 2. Mengisi Kelas Komponen

Buka `app/View/Components/NavMenu.php` dan ubah menjadi:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\Support\Facades\Request;

class NavMenu extends Component
{
    /**
     * Daftar item menu.
     * Setiap item adalah array dengan kunci:
     * - 'label': Teks yang ditampilkan
     * - 'route': Nama route (misalnya 'book.index')
     * - 'url': URL langsung (misalnya '/')
     * - 'activeWhen': Opsional, kondisi tambahan untuk menandai aktif (misalnya 'book.*')
     */
    public array $items;

    /**
     * Buat instance komponen.
     *
     * @param  array  $items  Daftar item menu
     */
    public function __construct(array $items = [])
    {
        $this->items = $items;
    }

    /**
     * Tentukan apakah item menu diberikan seharusnya dianggap aktif.
     *
     * @param  array  $item  Item menu
     * @return bool
     */
    public function isActive(array $item): bool
    {
        // Jika diberikan activeWhen khusus, gunakan itu
        if (!empty($item['activeWhen'])) {
            return request()->is($item['activeWhen']);
        }

        // Jika diberikan nama route, periksa apakah route saat ini cocok
        if (!empty($item['route'])) {
            return request()->routeIs($item['route']);
        }

        // Fallback: periksa URL yang diberikan (jika ada)
        if (!empty($item['url'])) {
            return request()->is($item['url'] . '*');
        }

        return false;
    }

    /**
     * Dapatkan tampilan / view yang mewakili komponen.
     *
     * @return \Illuminate\View\View|string
     */
    public function render()
    {
        return view('components.nav-menu');
    }
}
```

---

## 3. Mengisi Tampilan Komponen

Buka `resources/views/components/nav-menu.blade.php` dan ubah menjadi:

```blade
<nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
    <div class="container">
        <!-- Brand / Logo -->
        <a class="navbar-brand" href="{{ url('/') }}">📚 Perpustakaan Mini</a>

        <!-- Tombol toggler untuk layar kecil -->
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
                data-bs-target="#navbarNav" aria-controls="navbarNav"
                aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>

        <!-- Item menu -->
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav ms-auto">
                @foreach ($items as $item)
                    <li class="nav-item">
                        <a class="nav-link {{ $isActive($item) ? 'active' : '' }}"
                           href="{{ isset($item['url']) ? $item['url'] : route($item['route']) }}"
                           >
                            {{ $item['label'] }}
                        </a>
                    </li>
                @endforeach
            </ul>
        </div>
    </div>
</nav>
```

Catatan: Tampilan di atas menggunakan Bootstrap 5. Pastikan Anda telah memuat CSS Bootstrap di layout Anda (lihat langkah 5).

---

## 4. Menggunakan Komponen dalam Layout

Buka file layout Anda, misalnya `resources/views/layouts/app.blade.php`, dan tempatkan komponen navigasi di dalam `<body>` sebelum konten utama.

```blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>@yield('title', 'Aplikasi Laravel')</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
          rel="stylesheet">
</head>
<body>
    <!-- Tampilkan komponen navigasi -->
    <x-nav-menu :items="[
        ['label' => 'Beranda', 'url' => '/'],
        ['label' => 'Daftar Buku', 'route' => 'book.index'],
        ['label' => 'Tambah Buku', 'route' => 'book.create'],
        ['label' => 'Kategori', 'route' => 'category.index'],
    ]" />

    <main class="container">
        @if (session('success'))
            <div class="alert alert-success">{{ session('success') }}</div>
        @endif

        @yield('content')
    </main>

    <!-- Bootstrap JS dan dependensi (opsional, untuk fungsi dropdown dll.) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

Anda dapat menambahkan item menu lain sesuai kebutuhan aplikasi Anda.

---

## 5. Menambahkan Menu Dropdown (Opsional)

Jika Anda ingin menu dropdown, Anda dapat mengubah tampilan komponen menjadi lebih kompleks. Berikut contoh sederhana untuk menu dropdown "Pengguna":

Dalam array `items`, tambahkan item dengan kunci `children`:

```php
[
    ['label' => 'Pengguna', 'children' => [
        ['label' => 'Daftar Pengguna', 'route' => 'user.index'],
        ['label' => 'Tambah Pengguna', 'route' => 'user.create'],
    ]]
]
```

Kemudian ubah tampilan komponen untuk menangani atribut `children`:

```blade
@foreach ($items as $item)
    @if (isset($item['children']))
        <!-- Item menu dengan dropdown -->
        <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown{{ $loop->index }}"
               role="button" data-bs-toggle="dropdown" aria-expanded="false">
                {{ $item['label'] }}
            </a>
            <ul class="dropdown-menu" aria-labelledby="navbarDropdown{{ $loop->index }}">
                @foreach ($item['children'] as $child)
                    <li>
                        <a class="dropdown-item"
                           href="{{ isset($child['url']) ? $child['url'] : route($child['route']) }}">
                            {{ $child['label'] }}
                        </a>
                    </li>
                @endforeach
            </ul>
        </li>
    @else
        <!-- Item menu biasa -->
        <li class="nav-item">
            <a class="nav-link {{ $isActive($item) ? 'active' : '' }}"
               href="{{ isset($item['url']) ? $item['url'] : route($item['route']) }}">
                {{ $item['label'] }}
            </a>
        </li>
    @endif
@endforeach
```

---

## 6. Menyiapkan Item Menu Secara Dinamis (Opsional)

Alih-angkot menyusun array item menu secara manual di setiap layout, Anda dapat menggunakan view composer untuk menyediakan data menu ke semua view.

Contoh menggunakan layanan layanan (`App\Providers\ViewServiceProvider`):

```php
use Illuminate\Support\Facades\View;

public function boot(): void
{
    View::composer('*', function ($view) {
        $view->with('navMenuItems', [
            ['label' => 'Beranda', 'url' => '/'],
            ['label' => 'Daftar Buku', 'route' => 'book.index'],
            ['label' => 'Tambah Buku', 'route' => 'book.create'],
            ['label' => 'Kategori', 'route' => 'category.index'],
        ]);
    });
}
```

Dalam layout Anda, cukup gunakan variabel `$navMenuItems`:

```blade
<x-nav-menu :items="$navMenuItems" />
```

---

## 7. Ringkasan Perintah yang Digunakan

| Command                                                     | Fungsi                                   |
| ----------------------------------------------------------- | ---------------------------------------- |
| `php artisan make:component NavMenu`                        | Membuat komponen Blade `nav-menu`        |
| `php artisan serve`                                         | Menjalankan server pengembangan          |

---

## 8. Tantangan Lanjutan

Setelah komponen navigasi dasar berjalan, Anda bisa mencoba:
- Menambahkan ikon (misalnya menggunakan Font Awesome atau Bootstrap Icons) ke setiap item menu.
- Mengimplementasikan menu samping (sidebar) yang dapat ditutup/ dibuka.
- Menggunakan AJAX atau Laravel Livewire untuk memperbarui konten tanpa memuat ulang halaman.
- Menyimpan preferensi menu (misalnya negara bahasa) ke dalam session atau database.

---

Materi terkait: [Blade Components](https://laravel.com/docs/blade#components) · [Route Naming](https://laravel.com/docs/routing#named-routes) · [Request::is()](https://laravel.com/docs/requests#accessing-the-request) · [View Composers](https://laravel.com/docs/views#view-composers)