# CRUD dengan Relasi (One-to-Many)

Panduan ini menjelaskan cara menambahkan relasi **one-to-many** antara model `Book` (Buku) dan model `Category` (Kategori) ke dalam aplikasi CRUD yang sudah ada. Setiap buku appartenan ke satu kategori, sementara satu kategori bisa memiliki banyak buku.

!!! note "Prasyarat"
    Anda sudah memiliki aplikasi CRUD Buku yang berjalan dari tutorial [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md). Jika belum, silakan ikuti terlebih dahulu tutorial tersebut.

---

## 1. Membuat Model dan Migration untuk Category

Buat model beserta migration dan factory sekaligus:

```bash
php artisan make:model Category -mf
```

Buka file `database/migrations/xxxx_xx_xx_xxxxxx_create_categories_table.php`:

```php
public function up(): void
{
    Schema::create('categories', function (Blueprint $table) {
        $table->id();
        $table->string('name');
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

## 2. Menambahkan Kolom `category_id` ke Tabel `books`

Buat migration baru untuk menambahkan foreign key:

```bash
php artisan make:migration add_category_id_to_books_table --table=books
```

Buka file migrasi yang baru dibuat dan ubah menjadi:

```php
public function up(): void
{
    Schema::table('books', function (Blueprint $table) {
        $table->foreignId('category_id')->constrained()->onDelete('cascade');
    });
}

public function down(): void
{
    Schema::table('books', function (Blueprint $table) {
        $table->dropForeign(['category_id']);
        $table->dropColumn('category_id');
    });
}
```

Jalankan migration:

```bash
php artisan migrate
```

## 3. Mengupdate Model Book

Buka `app/Models/Book.php` dan tambahkan relasi:

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
        'category_id',
    ];

    public function category()
    {
        return $this->belongsTo(Category::class);
    }
}
```

## 4. Mengupdate Model Category

Buka `app/Models/Category.php` dan pastikan memiliki relasi hasMany:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function books()
    {
        return $this->hasMany(Book::class);
    }
}
```

## 5. Mengupdate Factory dan Seeder

### Category Factory
Buka `database/factories/CategoryFactory.php`:

```php
public function definition(): array
{
    return [
        'name' => fake()->word(),
    ];
}
```

### Book Factory (tambahkan category_id)
Buka `database/factories/BookFactory.php` dan ubah menjadi:

```php
public function definition(): array
{
    return [
        'title' => fake()->sentence(3),
        'writer' => fake()->name(),
        'publication_year' => fake()->numberBetween(1990, 2025),
        'description' => fake()->paragraph(),
        'category_id' => Category::factory(),
    ];
}
```

### DatabaseSeeder
Buka `database/seeders/DatabaseSeeder.php` dan tambahkan penyederhanaan kategori:

```php
use App\Models\Category;
use App\Models\Book;

public function run(): void
{
    // Buat 5 kategori terlebih dahulu
    Category::factory(5)->create();

    // Buat 25 buku dengan kategori acak
    Book::factory(25)->create();
}
```

Jalankan ulang seeder:

```bash
php artisan db:seed
```

## 6. Mengupdate Form Request (Validasi)

Buka `app/Http/Requests/BookRequest.php` dan tambahkan aturan untuk `category_id`:

```php
public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'writer' => ['required', 'string', 'max:255'],
        'publication_year' => ['required', 'integer', 'between:1000,' . date('Y')],
        'description' => ['nullable', 'string'],
        'category_id' => ['required', 'exists:categories,id'],
    ];
}

public function messages(): array
{
    return [
        'title.required' => 'Judul wajib diisi.',
        'writer.required' => 'Penulis wajib diisi.',
        'publication_year.required' => 'Tahun terbit wajib diisi.',
        'publication_year.integer' => 'Tahun terbit harus berupa angka.',
        'category_id.required' => 'Kategori wajib dipilih.',
        'category_id.exists' => 'Kategori yang dipilih tidak valid.',
    ];
}
```

## 7. Mengupdate Controller

Buka `app/Http/Controllers/BookController.php` dan pastikan method `index`, `show`, `store`, `update` bekerja dengan relasi (tidak perlu perubahan besar karena kita sudah memasukkan `category_id` dalam `fillable` dan menggunakan `validated()`). Namun kita perlu menyampaikan data kategori ke view.

### Tambahkan data kategori ke method `create` dan `edit`

```php
use App\Models\Category;

// ...

public function create()
{
    $categories = Category::all();
    return view('book.create', compact('categories'));
}

public function edit(Book $book)
{
    $categories = Category::all();
    return view('book.edit', compact('book', 'categories'));
}
```

Method `index` dan `show` sudah bisa menampilkan kategori melalui relasi, namun kita dapat menambahkan eager loading untuk efisiensi:

```php
public function index(Request $request)
{
    $books = Book::query()
        ->when($request->q, function ($query, $q) {
            $query->where('title', 'like', "%{$q}%")
                  ->orWhere('writer', 'like', "%{$q}%");
        })
        ->with('category') // eager loading
        ->latest()
        ->paginate(10)
        ->withQueryString();

    return view('book.index', compact('books'));
}

public function show(Book $book)
{
    $book->load('category'); // eager loading
    return view('book.show', compact($book));
}
```

Method `store` dan `update` tidak perlu diubah karena sudah menggunakan `$request->validated()` yang mencakup `category_id`.

## 8. Mengupdate Tampilan (Blade)

### 8.1 Layout dan partial tetap sama.

### 8.2 Daftar (`resources/views/book/index.blade.php`)

Tambahkan kolom Kategori di tabel:

```blade
<thead>
    <tr>
        <th>#</th>
        <th>Judul</th>
        <th>Penulis</th>
        <th>Tahun</th>
        <th>Kategori</th>
        <th class="text-end">Aksi</th>
    </tr>
</thead>

<td>{{ $item->category->name ?? '-' }}</td>
```

### 8.3 Form bersama (`resources/views/book/_form.blade.php`)

Tambahkan field select untuk kategori sebelum penutup form:

```blade
<div class="mb-3">
    <label for="category_id" class="form-label">Kategori</label>
    <select id="category_id" name="category_id"
            class="form-select @error('category_id') is-invalid @enderror">
        <option value="">-- Pilih Kategori --</option>
        @foreach ($categories as $category)
            <option value="{{ $category->id }}"
                    {{ old('category_id', $book->category_id ?? '') == $category->id ? 'selected' : '' }}
            >
                {{ $category->name }}
            </option>
        @endforeach
    </select>
    @error('category_id') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>
```

### 8.4 Detail (`resources/views/book/show.blade.php`)

Tambahkan baris untuk menampilkan kategori:

```blade
<p class="text-muted">
    {{ $book->writer }} &middot; {{ $book->publication_year }} &middot;
    {{ $book->category->name ?? 'Tanpa Kategori' }}
</p>
```

## 9. Jalankan dan Coba

```bash
php artisan serve
```

Uji fitur berikut:
- Daftar buku menampilkan kolom kategori.
- Form tambah/edit buku memiliki dropdown kategori yang terisi dari data kategori.
- Menyimpan buku dengan kategori yang dipilih.
- Halaman detail menunjukkan kategori buku.
- Pencarian masih berfungsi.
- Paginasi tetap works.

## 10. Ringkasan Command yang Dipakai

| Command                                                              | Fungsi                      |
| -------------------------------------------------------------------- | --------------------------- |
| `php artisan make:model Category -mf`                                | Model + migration + factory |
| `php artisan migrate`                                                | Membuat tabel categories    |
| `php artisan make:migration add_category_id_to_books_table`          | Menambahkan foreign key     |
| `php artisan migrate`                                                | Jalankan migration kolom    |
| `php artisan db:seed`                                                | Mengisi data kategori dan buku |
| `php artisan test`                                                   | Menjalankan semua test      |

## 11. Tantangan Lanjutan

Setelah relasi one-to-many berhasil, Anda bisa mencoba:
- Menambahkan relasi many-to-many (misalnya Buku dan Penulis).
- Menggunakan eager loading dengan `withCount()` untuk menghitung jumlah buku per kategori.
- Menambahkan filter berdasarkan kategori di halaman index.
- Membuat relasi polymorphics jika diperlukan.

---

Materi terkait: [File `.env`](env.md) · [Composer](composer.md) · [npm dan npx](npm.md) · [CRUD Data Buku](aplikasi-ujicoba-crud.md)