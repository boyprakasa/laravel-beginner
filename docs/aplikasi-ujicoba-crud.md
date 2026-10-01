# Aplikasi Ujicoba: CRUD Data Buku

Panduan ini membuat aplikasi kecil **Perpustakaan Mini** untuk mempraktikkan empat operasi dasar database:

| Huruf | Operasi              | Route                                   | Method controller |
| ----- | -------------------- | --------------------------------------- | ----------------- |
| **C** | Create (tambah data) | `GET /book/create`, `POST /book`        | `create`, `store` |
| **R** | Read (lihat data)    | `GET /book`, `GET /book/{id}`           | `index`, `show`   |
| **U** | Update (ubah data)   | `GET /book/{id}/edit`, `PUT /book/{id}` | `edit`, `update`  |
| **D** | Delete (hapus data)  | `DELETE /book/{id}`                     | `destroy`         |

Yang dipelajari: migration, model, factory, resource controller, validasi, Blade, flash message, pagination, pencarian, dan feature test.

!!! note "Prasyarat"
    Laravel dan PHP sudah terpasang (lihat panduan instalasi: [Windows](windows.md), [macOS](macos.md), atau [Linux](linux.md)).
    Database memakai **SQLite** bawaan Laravel, jadi tidak perlu install MySQL.
    Disarankan membaca dulu dasar-dasarnya: [File `.env`](env.md), [Composer](composer.md), dan [npm dan npx](npm.md).

---

## 1. Buat Project Baru

```bash
laravel new ujicoba-crud
```

Saat installer bertanya, pilih:

- Starter kit: **None**
- Testing framework: **Pest**
- Database: **SQLite**

Masuk ke folder project:

```bash
cd ujicoba-crud
```

Jika memakai Herd, folder project yang berada di direktori Herd otomatis tersedia di `http://ujicoba-crud.test`. Di Linux dengan Valet, letakkan project di folder yang sudah di-_park_ (lihat [panduan Linux](linux.md)). Tanpa keduanya, jalankan `php artisan serve`.

Penjelasan pilihan installer dan flag `laravel new` ada di bagian _Buat project_ pada panduan instalasi: [Windows](windows.md#14-buat-project), [macOS](macos.md#14-buat-project), atau [Linux](linux.md#111-buat-project).

---

## 2. Migration: Membuat Tabel

Buat model beserta migration dan factory sekaligus:

```bash
php artisan make:model Book -mf
```

Buka file `database/migrations/xxxx_xx_xx_xxxxxx_create_books_table.php`:

```php
public function up(): void
{
    Schema::create('books', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('writer');
        $table->unsignedSmallInteger('publication_year');
        $table->text('description')->nullable();
        $table->timestamps();
    });
}
```

Jalankan migration:

```bash
php artisan migrate
```

!!! tip "Referensi command"
    Penjelasan `make:model`, `migrate`, dan command Artisan lainnya ada di bagian _Command Artisan_ pada panduan instalasi: [Windows](windows.md#3-command-artisan), [macOS](macos.md#3-command-artisan), atau [Linux](linux.md#3-command-artisan).

---

## 3. Model

`app/Models/Book.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'writer',
        'publication_year',
        'description',
    ];
}
```

!!! info "Kenapa `$fillable`?"
Daftar ini menentukan kolom mana yang boleh diisi lewat `Book::create([...])`. Ini melindungi dari _mass assignment_, yaitu pengguna mengirim kolom yang seharusnya tidak boleh diubah.

---

## 4. Factory dan Seeder (Data Contoh)

`database/factories/BookFactory.php`

```php
public function definition(): array
{
    return [
        'title' => fake()->sentence(3),
        'writer' => fake()->name(),
        'publication_year' => fake()->numberBetween(1990, 2025),
        'description' => fake()->paragraph(),
    ];
}
```

`database/seeders/DatabaseSeeder.php`

```php
use App\Models\Book;

public function run(): void
{
    Book::factory(25)->create();
}
```

Isi data contoh:

```bash
php artisan db:seed
```

---

## 5. Validasi dengan Form Request

```bash
php artisan make:request BookRequest
```

`app/Http/Requests/BookRequest.php`

```php
public function authorize(): bool
{
    return true;
}

public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'writer' => ['required', 'string', 'max:255'],
        'publication_year' => ['required', 'integer', 'between:1000,' . date('Y')],
        'description' => ['nullable', 'string'],
    ];
}

public function messages(): array
{
    return [
        'title.required' => 'Judul wajib diisi.',
        'writer.required' => 'Penulis wajib diisi.',
        'publication_year.required' => 'Tahun terbit wajib diisi.',
        'publication_year.integer' => 'Tahun terbit harus berupa angka.',
    ];
}
```

---

## 6. Controller

```bash
php artisan make:controller BookController --resource --model=Book
```

`app/Http/Controllers/BookController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\BookRequest;
use App\Models\Book;
use Illuminate\Http\Request;

class BookController extends Controller
{
    // READ (daftar)
    public function index(Request $request)
    {
        $book = Book::query()
            ->when($request->q, function ($query, $q) {
                $query->where('title', 'like', "%{$q}%")
                      ->orWhere('writer', 'like', "%{$q}%");
            })
            ->latest()
            ->paginate(10)
            ->withQueryString();

        return view('book.index', compact('book'));
    }

    // CREATE (form)
    public function create()
    {
        return view('book.create');
    }

    // CREATE (simpan)
    public function store(BookRequest $request)
    {
        Book::create($request->validated());

        return redirect()->route('book.index')
            ->with('success', 'Buku berhasil ditambahkan.');
    }

    // READ (detail)
    public function show(Book $book)
    {
        return view('book.show', compact('book'));
    }

    // UPDATE (form)
    public function edit(Book $book)
    {
        return view('book.edit', compact('book'));
    }

    // UPDATE (simpan)
    public function update(BookRequest $request, Book $book)
    {
        $book->update($request->validated());

        return redirect()->route('book.index')
            ->with('success', 'Buku berhasil diperbarui.');
    }

    // DELETE
    public function destroy(Book $book)
    {
        $book->delete();

        return redirect()->route('book.index')
            ->with('success', 'Buku berhasil dihapus.');
    }
}
```

---

## 7. Route

`routes/web.php`

```php
use App\Http\Controllers\BookController;
use Illuminate\Support\Facades\Route;

Route::get('/', fn () => redirect()->route('book.index'));

Route::resource('book', BookController::class);
```

Satu baris `Route::resource` membuat 7 route sekaligus. Lihat daftarnya:

```bash
php artisan route:list --name=book
```

---

## 8. Tampilan (Blade)

Agar pagination tampil rapi dengan Bootstrap, tambahkan di `app/Providers/AppServiceProvider.php`:

```php
use Illuminate\Pagination\Paginator;

public function boot(): void
{
    Paginator::useBootstrapFive();
}
```

Buat folder dan file view:

```bash
mkdir -p resources/views/layouts resources/views/book
```

### 8.1 Layout: `resources/views/layouts/app.blade.php`

````blade
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>@yield('title', 'Perpustakaan Mini')</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <nav class="navbar navbar-dark bg-dark mb-4">
        <div class="container">
            <a class="navbar-brand" href="{{ route('book.index') }}">📚 Perpustakaan Mini</a>
        </div>
    </nav>

    <main class="container">
        @if (session('success'))
            <div class="alert alert-success">{{ session('success') }}</div>
        @endif

        @yield('content')
    </main>
</body>
</html>

- **Tip**: Untuk membuat navigasi yang dapat digunakan kembali, Anda dapat membuat komponen Blade seperti yang dijelaskan dalam [Komponen Blade: Navigasi Menu](komponen-navigasi.md).

### 8.2 Daftar: `resources/views/book/index.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Daftar Buku')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-3">
    <h1 class="h3 mb-0">Daftar Buku</h1>
    <a href="{{ route('book.create') }}" class="btn btn-primary">+ Tambah Buku</a>
</div>

<form method="GET" class="mb-3">
    <div class="input-group">
        <input type="text" name="q" value="{{ request('q') }}"
               class="form-control" placeholder="Cari judul atau penulis...">
        <button class="btn btn-outline-secondary">Cari</button>
    </div>
</form>

<div class="card">
    <div class="table-responsive">
        <table class="table table-striped mb-0">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Judul</th>
                    <th>Penulis</th>
                    <th>Tahun</th>
                    <th class="text-end">Aksi</th>
                </tr>
            </thead>
            <tbody>
                @forelse ($book as $item)
                    <tr>
                        <td>{{ $book->firstItem() + $loop->index }}</td>
                        <td>{{ $item->title }}</td>
                        <td>{{ $item->writer }}</td>
                        <td>{{ $item->publication_year }}</td>
                        <td class="text-end">
                            <a href="{{ route('book.show', $item) }}" class="btn btn-sm btn-info">Detail</a>
                            <a href="{{ route('book.edit', $item) }}" class="btn btn-sm btn-warning">Edit</a>
                            <form action="{{ route('book.destroy', $item) }}" method="POST"
                                  class="d-inline"
                                  onsubmit="return confirm('Yakin hapus buku ini?')">
                                @csrf
                                @method('DELETE')
                                <button class="btn btn-sm btn-danger">Hapus</button>
                            </form>
                        </td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="5" class="text-center text-muted py-4">Belum ada data.</td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>
</div>

<div class="mt-3">
    {{ $book->links() }}
</div>
@endsection
````

### 8.3 Form bersama: `resources/views/book/_form.blade.php`

```blade
<div class="mb-3">
    <label for="title" class="form-label">Judul</label>
    <input type="text" id="title" name="title"
           value="{{ old('title', $book->title ?? '') }}"
           class="form-control @error('title') is-invalid @enderror">
    @error('title') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>

<div class="mb-3">
    <label for="writer" class="form-label">Penulis</label>
    <input type="text" id="writer" name="writer"
           value="{{ old('writer', $book->writer ?? '') }}"
           class="form-control @error('writer') is-invalid @enderror">
    @error('writer') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>

<div class="mb-3">
    <label for="publication_year" class="form-label">Tahun Terbit</label>
    <input type="number" id="publication_year" name="publication_year"
           value="{{ old('publication_year', $book->publication_year ?? '') }}"
           class="form-control @error('publication_year') is-invalid @enderror">
    @error('publication_year') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>

<div class="mb-3">
    <label for="description" class="form-label">Deskripsi</label>
    <textarea id="description" name="description" rows="4"
              class="form-control @error('description') is-invalid @enderror">{{ old('description', $book->description ?? '') }}</textarea>
    @error('description') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>
```

### 8.4 Tambah: `resources/views/book/create.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Tambah Buku')

@section('content')
<h1 class="h3 mb-3">Tambah Buku</h1>

<div class="card">
    <div class="card-body">
        <form action="{{ route('book.store') }}" method="POST">
            @csrf
            @include('book._form')
            <button class="btn btn-primary">Simpan</button>
            <a href="{{ route('book.index') }}" class="btn btn-secondary">Batal</a>
        </form>
    </div>
</div>
@endsection
```

### 8.5 Edit: `resources/views/book/edit.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Edit Buku')

@section('content')
<h1 class="h3 mb-3">Edit Buku</h1>

<div class="card">
    <div class="card-body">
        <form action="{{ route('book.update', $book) }}" method="POST">
            @csrf
            @method('PUT')
            @include('book._form')
            <button class="btn btn-primary">Perbarui</button>
            <a href="{{ route('book.index') }}" class="btn btn-secondary">Batal</a>
        </form>
    </div>
</div>
@endsection
```

### 8.6 Detail: `resources/views/book/show.blade.php`

```blade
@extends('layouts.app')

@section('title', $book->title)

@section('content')
<div class="card">
    <div class="card-body">
        <h1 class="h3">{{ $book->title }}</h1>
        <p class="text-muted">
            {{ $book->writer }} &middot; {{ $book->publication_year }}
        </p>
        <p>{{ $book->description ?: 'Tidak ada deskripsi.' }}</p>
        <a href="{{ route('book.index') }}" class="btn btn-secondary">Kembali</a>
    </div>
</div>
@endsection
```

---

## 9. Jalankan dan Coba Manual

```bash
php artisan serve
```

Buka `http://localhost:8000` (atau `http://ujicoba-crud.test` bila memakai Herd), lalu coba:

- [ ] Data contoh tampil di daftar, 10 per halaman
- [ ] Pencarian judul/penulis berfungsi
- [ ] Tambah buku baru berhasil, muncul pesan sukses
- [ ] Kirim form kosong, muncul pesan error di bawah kolom
- [ ] Edit buku, data lama terisi otomatis di form
- [ ] Halaman detail menampilkan data yang benar
- [ ] Hapus buku menampilkan konfirmasi, lalu data hilang

!!! tip "Tidak perlu `npm run dev`"
Aplikasi ini memakai Bootstrap dari CDN, jadi tidak membutuhkan Vite. Kalau nanti kamu mengelola CSS dan JS sendiri, lihat [npm dan npx](npm.md). Pengaturan database (SQLite) tersimpan di file `.env`, penjelasannya ada di [File `.env`](env.md).

---

## 10. Feature Test Otomatis

Buat file test:

```bash
php artisan make:test BookTest --pest
```

`tests/Feature/BookTest.php`

```php
<?php

use App\Models\Book;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

$dataValid = [
    'title' => 'Laravel untuk Pemula',
    'writer' => 'Budi Santoso',
    'publication_year' => 2024,
    'description' => 'Belajar Laravel dari nol.',
];

it('menampilkan daftar buku', function () {
    Book::factory()->count(3)->create();

    $this->get(route('book.index'))
        ->assertOk()
        ->assertViewHas('book');
});

it('dapat menambah buku', function () use ($dataValid) {
    $this->post(route('book.store'), $dataValid)
        ->assertRedirect(route('book.index'));

    $this->assertDatabaseHas('books', ['title' => 'Laravel untuk Pemula']);
});

it('menolak data tidak valid saat menambah', function () {
    $this->post(route('book.store'), [])
        ->assertSessionHasErrors(['title', 'writer', 'publication_year']);
});

it('menampilkan detail buku', function () {
    $book = Book::factory()->create();

    $this->get(route('book.show', $book))
        ->assertOk()
        ->assertSee($book->title);
});

it('dapat memperbarui buku', function () use ($dataValid) {
    $book = Book::factory()->create();

    $this->put(route('book.update', $book), $dataValid)
        ->assertRedirect(route('book.index'));

    $this->assertDatabaseHas('books', [
        'id' => $book->id,
        'title' => 'Laravel untuk Pemula',
    ]);
});

it('dapat menghapus buku', function () {
    $book = Book::factory()->create();

    $this->delete(route('book.destroy', $book))
        ->assertRedirect(route('book.index'));

    $this->assertDatabaseMissing('books', ['id' => $book->id]);
});
```

Jalankan:

```bash
php artisan test
```

Semua test harus hijau (_passed_).

---

## 11. Ringkasan Command yang Dipakai

| Command                                                              | Fungsi                      |
| -------------------------------------------------------------------- | --------------------------- |
| `laravel new ujicoba-crud`                                           | Membuat project baru        |
| `php artisan make:model Book -mf`                                    | Model + migration + factory |
| `php artisan migrate`                                                | Membuat tabel di database   |
| `php artisan db:seed`                                                | Mengisi data contoh         |
| `php artisan make:request BookRequest`                               | Membuat kelas validasi      |
| `php artisan make:controller BookController --resource --model=Book` | Controller CRUD lengkap     |
| `php artisan route:list --name=book`                                 | Melihat daftar route        |
| `php artisan make:test BookTest --pest`                              | Membuat file test           |
| `php artisan test`                                                   | Menjalankan semua test      |

Untuk mengulang dari awal dengan data bersih:

```bash
php artisan migrate:fresh --seed
```

!!! warning "Hati-hati"
    `migrate:fresh` **menghapus semua tabel** lalu membuatnya ulang. Pakai hanya di lingkungan belajar/pengembangan, jangan di database produksi.

---

## 12. Tantangan Lanjutan

Setelah CRUD dasar jalan, coba kembangkan:

1. Tambah kolom `stok` dan `kategori`, lalu buat migration baru dengan `php artisan make:migration add_stok_to_books_table`.
2. Tambah upload gambar sampul buku (`Storage` dan `php artisan storage:link`).
3. Ganti hapus biasa menjadi **soft delete** (`SoftDeletes`).
4. Tambah login dengan Laravel Breeze agar hanya pengguna terdaftar yang bisa mengubah data. Cara memasang package dijelaskan di [Composer](composer.md).
5. Buat relasi Kategori dan Buku (one-to-many) - [CRUD dengan Relasi](crud-relasi.md)

---

Materi terkait: [File `.env`](env.md) · [Composer](composer.md) · [npm dan npx](npm.md)
