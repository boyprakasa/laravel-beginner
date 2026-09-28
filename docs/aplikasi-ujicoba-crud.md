# Aplikasi Ujicoba: CRUD Data Buku

Panduan ini membuat aplikasi kecil **Perpustakaan Mini** untuk mempraktikkan empat operasi dasar database:

| Huruf | Operasi | Route | Method controller |
|-------|---------|-------|-------------------|
| **C** | Create (tambah data) | `GET /buku/create`, `POST /buku` | `create`, `store` |
| **R** | Read (lihat data) | `GET /buku`, `GET /buku/{id}` | `index`, `show` |
| **U** | Update (ubah data) | `GET /buku/{id}/edit`, `PUT /buku/{id}` | `edit`, `update` |
| **D** | Delete (hapus data) | `DELETE /buku/{id}` | `destroy` |

Yang dipelajari: migration, model, factory, resource controller, validasi, Blade, flash message, pagination, pencarian, dan feature test.

!!! note "Prasyarat"
    Laravel dan PHP sudah terpasang (lihat panduan instalasi: [Windows](windows.md), [macOS](macos.md), atau [Linux](linux.md)). Database memakai **SQLite** bawaan Laravel, jadi tidak perlu install MySQL.

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

Jika memakai Herd, folder project yang berada di direktori Herd otomatis tersedia di `http://ujicoba-crud.test`. Di Linux dengan Valet, letakkan project di folder yang sudah di-*park* (lihat [panduan Linux](linux.md)). Tanpa keduanya, jalankan `php artisan serve`.

Penjelasan pilihan installer dan flag `laravel new` ada di bagian *Buat project* pada panduan instalasi: [Windows](windows.md#14-buat-project), [macOS](macos.md#14-buat-project), atau [Linux](linux.md#111-buat-project).

---

## 2. Migration: Membuat Tabel

Buat model beserta migration dan factory sekaligus:

```bash
php artisan make:model Buku -mf
```

Buka file `database/migrations/xxxx_xx_xx_xxxxxx_create_bukus_table.php`:

```php
public function up(): void
{
    Schema::create('bukus', function (Blueprint $table) {
        $table->id();
        $table->string('judul');
        $table->string('penulis');
        $table->unsignedSmallInteger('tahun_terbit');
        $table->text('deskripsi')->nullable();
        $table->timestamps();
    });
}
```

Jalankan migration:

```bash
php artisan migrate
```

!!! tip "Referensi command"
    Penjelasan `make:model`, `migrate`, dan command Artisan lainnya ada di bagian *Command Artisan* pada panduan instalasi: [Windows](windows.md#3-command-artisan), [macOS](macos.md#3-command-artisan), atau [Linux](linux.md#3-command-artisan).

---

## 3. Model

`app/Models/Buku.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Buku extends Model
{
    use HasFactory;

    protected $fillable = [
        'judul',
        'penulis',
        'tahun_terbit',
        'deskripsi',
    ];
}
```

!!! info "Kenapa `$fillable`?"
    Daftar ini menentukan kolom mana yang boleh diisi lewat `Buku::create([...])`. Ini melindungi dari *mass assignment*, yaitu pengguna mengirim kolom yang seharusnya tidak boleh diubah.

---

## 4. Factory dan Seeder (Data Contoh)

`database/factories/BukuFactory.php`

```php
public function definition(): array
{
    return [
        'judul' => fake()->sentence(3),
        'penulis' => fake()->name(),
        'tahun_terbit' => fake()->numberBetween(1990, 2025),
        'deskripsi' => fake()->paragraph(),
    ];
}
```

`database/seeders/DatabaseSeeder.php`

```php
use App\Models\Buku;

public function run(): void
{
    Buku::factory(25)->create();
}
```

Isi data contoh:

```bash
php artisan db:seed
```

---

## 5. Validasi dengan Form Request

```bash
php artisan make:request BukuRequest
```

`app/Http/Requests/BukuRequest.php`

```php
public function authorize(): bool
{
    return true;
}

public function rules(): array
{
    return [
        'judul' => ['required', 'string', 'max:255'],
        'penulis' => ['required', 'string', 'max:255'],
        'tahun_terbit' => ['required', 'integer', 'between:1000,' . date('Y')],
        'deskripsi' => ['nullable', 'string'],
    ];
}

public function messages(): array
{
    return [
        'judul.required' => 'Judul wajib diisi.',
        'penulis.required' => 'Penulis wajib diisi.',
        'tahun_terbit.required' => 'Tahun terbit wajib diisi.',
        'tahun_terbit.integer' => 'Tahun terbit harus berupa angka.',
    ];
}
```

---

## 6. Controller

```bash
php artisan make:controller BukuController --resource --model=Buku
```

`app/Http/Controllers/BukuController.php`

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\BukuRequest;
use App\Models\Buku;
use Illuminate\Http\Request;

class BukuController extends Controller
{
    // READ (daftar)
    public function index(Request $request)
    {
        $buku = Buku::query()
            ->when($request->q, function ($query, $q) {
                $query->where('judul', 'like', "%{$q}%")
                      ->orWhere('penulis', 'like', "%{$q}%");
            })
            ->latest()
            ->paginate(10)
            ->withQueryString();

        return view('buku.index', compact('buku'));
    }

    // CREATE (form)
    public function create()
    {
        return view('buku.create');
    }

    // CREATE (simpan)
    public function store(BukuRequest $request)
    {
        Buku::create($request->validated());

        return redirect()->route('buku.index')
            ->with('success', 'Buku berhasil ditambahkan.');
    }

    // READ (detail)
    public function show(Buku $buku)
    {
        return view('buku.show', compact('buku'));
    }

    // UPDATE (form)
    public function edit(Buku $buku)
    {
        return view('buku.edit', compact('buku'));
    }

    // UPDATE (simpan)
    public function update(BukuRequest $request, Buku $buku)
    {
        $buku->update($request->validated());

        return redirect()->route('buku.index')
            ->with('success', 'Buku berhasil diperbarui.');
    }

    // DELETE
    public function destroy(Buku $buku)
    {
        $buku->delete();

        return redirect()->route('buku.index')
            ->with('success', 'Buku berhasil dihapus.');
    }
}
```

---

## 7. Route

`routes/web.php`

```php
use App\Http\Controllers\BukuController;
use Illuminate\Support\Facades\Route;

Route::get('/', fn () => redirect()->route('buku.index'));

Route::resource('buku', BukuController::class);
```

Satu baris `Route::resource` membuat 7 route sekaligus. Lihat daftarnya:

```bash
php artisan route:list --name=buku
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
mkdir -p resources/views/layouts resources/views/buku
```

### 8.1 Layout: `resources/views/layouts/app.blade.php`

```blade
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
            <a class="navbar-brand" href="{{ route('buku.index') }}">📚 Perpustakaan Mini</a>
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
```

### 8.2 Daftar: `resources/views/buku/index.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Daftar Buku')

@section('content')
<div class="d-flex justify-content-between align-items-center mb-3">
    <h1 class="h3 mb-0">Daftar Buku</h1>
    <a href="{{ route('buku.create') }}" class="btn btn-primary">+ Tambah Buku</a>
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
                @forelse ($buku as $item)
                    <tr>
                        <td>{{ $buku->firstItem() + $loop->index }}</td>
                        <td>{{ $item->judul }}</td>
                        <td>{{ $item->penulis }}</td>
                        <td>{{ $item->tahun_terbit }}</td>
                        <td class="text-end">
                            <a href="{{ route('buku.show', $item) }}" class="btn btn-sm btn-info">Detail</a>
                            <a href="{{ route('buku.edit', $item) }}" class="btn btn-sm btn-warning">Edit</a>
                            <form action="{{ route('buku.destroy', $item) }}" method="POST"
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
    {{ $buku->links() }}
</div>
@endsection
```

### 8.3 Form bersama: `resources/views/buku/_form.blade.php`

```blade
<div class="mb-3">
    <label for="judul" class="form-label">Judul</label>
    <input type="text" id="judul" name="judul"
           value="{{ old('judul', $buku->judul ?? '') }}"
           class="form-control @error('judul') is-invalid @enderror">
    @error('judul') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>

<div class="mb-3">
    <label for="penulis" class="form-label">Penulis</label>
    <input type="text" id="penulis" name="penulis"
           value="{{ old('penulis', $buku->penulis ?? '') }}"
           class="form-control @error('penulis') is-invalid @enderror">
    @error('penulis') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>

<div class="mb-3">
    <label for="tahun_terbit" class="form-label">Tahun Terbit</label>
    <input type="number" id="tahun_terbit" name="tahun_terbit"
           value="{{ old('tahun_terbit', $buku->tahun_terbit ?? '') }}"
           class="form-control @error('tahun_terbit') is-invalid @enderror">
    @error('tahun_terbit') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>

<div class="mb-3">
    <label for="deskripsi" class="form-label">Deskripsi</label>
    <textarea id="deskripsi" name="deskripsi" rows="4"
              class="form-control @error('deskripsi') is-invalid @enderror">{{ old('deskripsi', $buku->deskripsi ?? '') }}</textarea>
    @error('deskripsi') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>
```

### 8.4 Tambah: `resources/views/buku/create.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Tambah Buku')

@section('content')
<h1 class="h3 mb-3">Tambah Buku</h1>

<div class="card">
    <div class="card-body">
        <form action="{{ route('buku.store') }}" method="POST">
            @csrf
            @include('buku._form')
            <button class="btn btn-primary">Simpan</button>
            <a href="{{ route('buku.index') }}" class="btn btn-secondary">Batal</a>
        </form>
    </div>
</div>
@endsection
```

### 8.5 Edit: `resources/views/buku/edit.blade.php`

```blade
@extends('layouts.app')

@section('title', 'Edit Buku')

@section('content')
<h1 class="h3 mb-3">Edit Buku</h1>

<div class="card">
    <div class="card-body">
        <form action="{{ route('buku.update', $buku) }}" method="POST">
            @csrf
            @method('PUT')
            @include('buku._form')
            <button class="btn btn-primary">Perbarui</button>
            <a href="{{ route('buku.index') }}" class="btn btn-secondary">Batal</a>
        </form>
    </div>
</div>
@endsection
```

### 8.6 Detail: `resources/views/buku/show.blade.php`

```blade
@extends('layouts.app')

@section('title', $buku->judul)

@section('content')
<div class="card">
    <div class="card-body">
        <h1 class="h3">{{ $buku->judul }}</h1>
        <p class="text-muted">
            {{ $buku->penulis }} &middot; {{ $buku->tahun_terbit }}
        </p>
        <p>{{ $buku->deskripsi ?: 'Tidak ada deskripsi.' }}</p>
        <a href="{{ route('buku.index') }}" class="btn btn-secondary">Kembali</a>
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
php artisan make:test BukuTest --pest
```

`tests/Feature/BukuTest.php`

```php
<?php

use App\Models\Buku;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

$dataValid = [
    'judul' => 'Laravel untuk Pemula',
    'penulis' => 'Budi Santoso',
    'tahun_terbit' => 2024,
    'deskripsi' => 'Belajar Laravel dari nol.',
];

it('menampilkan daftar buku', function () {
    Buku::factory()->count(3)->create();

    $this->get(route('buku.index'))
        ->assertOk()
        ->assertViewHas('buku');
});

it('dapat menambah buku', function () use ($dataValid) {
    $this->post(route('buku.store'), $dataValid)
        ->assertRedirect(route('buku.index'));

    $this->assertDatabaseHas('bukus', ['judul' => 'Laravel untuk Pemula']);
});

it('menolak data tidak valid saat menambah', function () {
    $this->post(route('buku.store'), [])
        ->assertSessionHasErrors(['judul', 'penulis', 'tahun_terbit']);
});

it('menampilkan detail buku', function () {
    $buku = Buku::factory()->create();

    $this->get(route('buku.show', $buku))
        ->assertOk()
        ->assertSee($buku->judul);
});

it('dapat memperbarui buku', function () use ($dataValid) {
    $buku = Buku::factory()->create();

    $this->put(route('buku.update', $buku), $dataValid)
        ->assertRedirect(route('buku.index'));

    $this->assertDatabaseHas('bukus', [
        'id' => $buku->id,
        'judul' => 'Laravel untuk Pemula',
    ]);
});

it('dapat menghapus buku', function () {
    $buku = Buku::factory()->create();

    $this->delete(route('buku.destroy', $buku))
        ->assertRedirect(route('buku.index'));

    $this->assertDatabaseMissing('bukus', ['id' => $buku->id]);
});
```

Jalankan:

```bash
php artisan test
```

Semua test harus hijau (*passed*).

---

## 11. Ringkasan Command yang Dipakai

| Command | Fungsi |
|---------|--------|
| `laravel new ujicoba-crud` | Membuat project baru |
| `php artisan make:model Buku -mf` | Model + migration + factory |
| `php artisan migrate` | Membuat tabel di database |
| `php artisan db:seed` | Mengisi data contoh |
| `php artisan make:request BukuRequest` | Membuat kelas validasi |
| `php artisan make:controller BukuController --resource --model=Buku` | Controller CRUD lengkap |
| `php artisan route:list --name=buku` | Melihat daftar route |
| `php artisan make:test BukuTest --pest` | Membuat file test |
| `php artisan test` | Menjalankan semua test |

Untuk mengulang dari awal dengan data bersih:

```bash
php artisan migrate:fresh --seed
```

!!! warning "Hati-hati"
    `migrate:fresh` **menghapus semua tabel** lalu membuatnya ulang. Pakai hanya di lingkungan belajar/pengembangan, jangan di database produksi.

---

## 12. Tantangan Lanjutan

Setelah CRUD dasar jalan, coba kembangkan:

1. Tambah kolom `stok` dan `kategori`, lalu buat migration baru dengan `php artisan make:migration add_stok_to_bukus_table`.
2. Tambah upload gambar sampul buku (`Storage` dan `php artisan storage:link`).
3. Ganti hapus biasa menjadi **soft delete** (`SoftDeletes`).
4. Tambah login dengan Laravel Breeze agar hanya pengguna terdaftar yang bisa mengubah data. Cara memasang package dijelaskan di [Composer](composer.md).
5. Buat relasi `Kategori` dan `Buku` (one-to-many).

---

Materi terkait: [File `.env`](env.md) · [Composer](composer.md) · [npm dan npx](npm.md)
