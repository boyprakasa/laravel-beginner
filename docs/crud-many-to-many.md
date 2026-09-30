# CRUD dengan Relasi Many-to-Many (Penulis)

Panduan ini menjelaskan cara mengganti campo **writer** (string) pada model `Book` menjadi relasi **many-to-many** dengan model `Author` (Penulis). Setelah perubahan ini, satu buku dapat memiliki banyak penulis, dan satu penulis dapat menulis banyak buku.

!!! note "Prasyarat"
    Anda sudah memiliki aplikasi CRUD Buku yang berjalan dari tutorial [Aplikasi Uji Coba: CRUD Data Buku](aplikasi-ujicoba-crud.md). Jika belum, silakan ikuti terlebih dahulu tutorial tersebut.
    Panduan ini mengasumsikan Anda siap mengubah struktur tabel `books` (menghapus kolom `writer` dan menambahkan relasi many-to-many ke `authors`).

---

## 1. Membuat Model dan Migration untuk Author

Buat model beserta migration dan factory sekaligus:

```bash
php artisan make:model Author -mf
```

Buka file `database/migrations/xxxx_xx_xx_xxxxxx_create_authors_table.php`:

```php
public function up(): void
{
    Schema::create('authors', function (Blueprint $table) {
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

## 2. Menghapus Kolom `writer` dari Tabel `books`

Karena kita akan mengganti campo `writer` dengan relasi ke `Author`, kita perlu menghapus kolom `writer` dari tabel `books`.

Buat migration untuk menghapus kolom:

```bash
php artisan make:migration remove_writer_from_books_table --table=books
```

Buka file migrasi yang baru dibuat dan ubah menjadi:

```php
public function up(): void
{
    Schema::table('books', function (Blueprint $table) {
        $table->dropColumn('writer');
    });
}

public function down(): void
{
    Schema::table('books', function (Blueprint $table) {
        $table->string('writer')->after('title')->nullable();
    });
}
```

Jalankan migration:

```bash
php artisan migrate
```

## 3. Membuat Migration Pivot Table

Buat migration untuk tabel pivot yang akan menghubungkan buku dan penulis:

```bash
php artisan make:migration create_book_author_table
```

Buka file migrasi yang baru dibuat dan ubah menjadi:

```php
public function up(): void
{
    Schema::create('book_author', function (Blueprint $table) {
        $table->id();
        $table->foreignId('book_id')->constrained()->onDelete('cascade');
        $table->foreignId('author_id')->constrained()->onDelete('cascade');
        $table->timestamps();

        $table->unique(['book_id', 'author_id']);
    });
}

public function down(): void
{
    Schema::dropIfExists('book_author');
}
```

Jalankan migration:

```bash
php artisan migrate
```

## 4. Mengupdate Model Book

Buka `app/Models/Book.php` dan ubah menjadi:

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
        'publication_year',
        'description',
        'category_id',
    ];

    // Relasi one-to-many ke Category (jika sudah ada)
    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    // Relasi many-to-many ke Author
    public function authors()
    {
        return $this->belongsToMany(Author::class);
    }
}
```

## 5. Mengupdate Model Author

Buka `app/Models/Author.php` dan pastikan memiliki relasi belongsToMany:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Author extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    // Relasi many-to-many ke Book
    public function books()
    {
        return $this->belongsToMany(Book::class);
    }
}
```

## 6. Mengupdate Factory dan Seeder

### Author Factory
Buka `database/factories/AuthorFactory.php`:

```php
public function definition(): array
{
    return [
        'name' => fake()->name(),
    ];
}
```

### Book Factory (hapus writer)
Buka `database/factories/BookFactory.php` dan ubah menjadi:

```php
public function definition(): array
{
    return [
        'title' => fake()->sentence(3),
        'publication_year' => fake()->numberBetween(1990, 2025),
        'description' => fake()->paragraph(),
        'category_id' => Category::factory(),
    ];
}
```

### DatabaseSeeder
Buka `database/seeders/DatabaseSeeder.php` dan ubah menjadi:

```php
use App\Models\Author;
use App\Models\Book;
use App\Models\Category;

public function run(): void
{
    // Buat 5 kategori
    Category::factory(5)->create();

    // Buat 10 penulis
    Author::factory(10)->create();

    // Buat 25 buku dan asignakan penulis secara acak
    Book::factory(25)->create()->each(function ($book) {
        // Pilih 1-3 penulis secara acak untuk setiap buku
        $book->authors()->attach(
            Author::inRandomOrder()->limit(rand(1, 3))->pluck('id')
        );
    });
}
```

Jalankan ulang seeder:

```bash
php artisan db:seed
```

## 7. Mengupdate Form Request (Validasi)

Buka `app/Http/Requests/BookRequest.php` dan ubah aturan: hapus `writer`, pastikan tidak ada validasi writer.

```php
public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'publication_year' => ['required', 'integer', 'between:1000,' . date('Y')],
        'description' => ['nullable', 'string'],
        'category_id' => ['required', 'exists:categories,id'],
        'author_ids' => ['array'],
        'author_ids.*' => ['exists:authors,id'],
    ];
}

public function messages(): array
{
    return [
        'title.required' => 'Judul wajib diisi.',
        'publication_year.required' => 'Tahun terbit wajib diisi.',
        'publication_year.integer' => 'Tahun terbit harus berupa angka.',
        'category_id.required' => 'Kategori wajib dipilih.',
        'category_id.exists' => 'Kategori yang dipilih tidak valid.',
        'author_ids.*.exists' => 'Penulis yang dipilih tidak valid.',
    ];
}
```

## 8. Mengupdate Controller

Buka `app/Http/Controllers/BookController.php` dan ubah method `store` dan `update` untuk menghapus penulisan `writer` dan menyinkronkan penulis.

Tambahkan use statement untuk Author jika belum ada:

```php
use App\Models\Author;
```

### Method store
Ubah menjadi:

```php
public function store(BookRequest $request)
{
    $validated = $request->validated();

    // Ambil author_ids sebelum membuat buku
    $authorIds = $validated['author_ids'] ?? [];
    unset($validated['author_ids']);

    $book = Book::create($validated);

    // Sinkronkan penulis
    $book->authors()->sync($authorIds);

    return redirect()->route('book.index')
        ->with('success', 'Buku berhasil ditambahkan.');
}
```

### Method update
Ubah menjadi:

```php
public function update(BookRequest $request, Book $book)
{
    $validated = $request->validated();

    // Ambil author_ids sebelum memperbarui buku
    $authorIds = $validated['author_ids'] ?? [];
    unset($validated['author_ids']);

    $book->update($validated);

    // Sinkronkan penulis
    $book->authors()->sync($authorIds);

    return redirect()->route('book.index')
        ->with('success', 'Buku berhasil diperbarui.');
}
```

Method `index`, `show`, `create`, `edit` tidak perlu diubah, kecuali kita ingin menambahkan eager loading untuk author:

```php
// Di method index
$books = Book::query()
    ->when($request->q, function ($query, $q) {
        $query->where('title', 'like', "%{$q}%")
              ->orWhere('writer', 'like', "%{$q}%"); // Note: writer column sudah dihapus, baris ini akan menyebabkan error.
    })
    ->with(['category', 'authors']) // eager loading category dan authors
    ->latest()
    ->paginate(10)
    ->withQueryString();

return view('book.index', compact('books'));
```

Karena kolom `writer` sudah dihapus, kita perlu menghapus referensi ke `writer` di method `index`. Ubah menjadi:

```php
public function index(Request $request)
{
    $books = Book::query()
        ->when($request->q, function ($query, $q) {
            $query->where('title', 'like', "%{$q}%");
            // Jika masih ingin mencari berdasarkan nama penulis, kita perlu join dengan authors:
            ->orWhereHas('authors', function ($query) use ($q) {
                $query->where('name', 'like', "%{$q}%");
            });
        })
        ->with(['category', 'authors']) // eager loading category dan authors
        ->latest()
        ->paginate(10)
        ->withQueryString();

    return view('book.index', compact('books'));
}

public function show(Book $book)
{
    $book->load(['category', 'authors']);
    return view('book.show', compact('book'));
}
```

## 9. Mengupdate Tampilan (Blade)

### 9.1 Daftar (`resources/views/book/index.blade.php`)

Ubah kolom penulis menjadi menampilkan nama-nama penulis dari relasi authors. Karena kolom `writer` sudah tidak ada, kita hapus pencarian berdasarkan writer dan ganti dengan pencarian berdasarkan nama penulis (optional). Untuk kesederhanaan, kita bisa hanya menampilkan penulis dan menghapus pencarian writer, atau menambah pencarian penulis seperti di atas.

Kita akan memperbarui kolom Judul dan menambahkan kolom Penulis seperti ini:

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

<td>
    @foreach ($book->authors as $author)
        <span class="badge bg-secondary me-1">{{ $author->name }}</span>
    @endforeach
</td>
```

Untuk pencarian, kita bisa mengganti baris pada method `index` seperti di atas (jika ingin mencari berdasarkan nama penulis). Jika tidak, kita bisa hanya mencari berdasarkan judul.

### 9.2 Form bersama (`resources/views/book/_form.blade.php`)

Hapus field input untuk `writer` dan ganti dengan field checkbox untuk penulis (sama seperti sebelumnya):

```blade
<div class="mb-3">
    <label class="form-label">Penulis</label>
    <div>
        @foreach ($authors as $author)
            <div class="form-check">
                <input class="form-check-input"
                       type="checkbox"
                       name="author_ids[]"
                       value="{{ $author->id }}"
                       {{ in_array($author->id, old('author_ids', $book->authors->pluck('id')->toArray())) ? 'checked' : '' }}
                >
                <label class="form-check-label">{{ $author->name }}</label>
            </div>
        @endforeach
    </div>
    @error('author_ids') <div class="invalid-feedback">{{ $message }}</div> @enderror
</div>
```

### 9.3 Detail (`resources/views/book/show.blade.php`)

Ubah baris penulis menjadi menampilkan daftar penulis:

```blade
<p class="text-muted">
    {{ $book->publication_year }} &middot;
    {{ $book->category->name ?? 'Tanpa Kategori' }}
</p>

<div class="mt-3">
    <h5>Penulis</h5>
    @if ($book->authors->isNotEmpty())
        <div>
            @foreach ($book->authors as $author)
                <span class="badge bg-secondary me-1">{{ $author->name }}</span>
            @endforeach
        </div>
    @else
        <span class="text-muted">Belum ada penulis.</span>
    @endif
</div>
```

## 10. Mengirim Data Penulis ke View

Dalam method `create` dan `edit` controller, kirim data penulis ke view:

```php
use App\Models\Author;

// ...

public function create()
{
    $categories = Category::all();
    $authors = Author::orderBy('name')->get();
    return view('book.create', compact('categories', 'authors'));
}

public function edit(Book $book)
{
    $categories = Category::all();
    $authors = Author::orderBy('name')->get();
    return view('book.edit', compact('book', 'categories', 'authors'));
}
```

## 11. Jalankan dan Coba

```bash
php artisan serve
```

Uji fitur berikut:
- Daftar buku tidak lagi menampilkan kolom writer, sondern menampilkan kolom Penulis dengan badge-badgenya.
- Form tambah/edit buku tidak lagi memiliki field Writer, melainkan daftar checkbox penulis yang dapat dipilih banyak.
- Menyimpan buku dengan penulis yang dipilih.
- Halaman detail menunjukkan daftar penulis buku.
- Pencarian berdasarkan judul tetap berfungsi; jika Anda mengaktifkan pencarian berdasarkan nama penulis (lihat catatan di method index), maka pencarian juga akan mencari penulis.
- Paginasi tetap berfungsi.

## 12. Ringkasan Command yang Dipakai

| Command                                                              | Fungsi                      |
| -------------------------------------------------------------------- | --------------------------- |
| `php artisan make:model Author -mf`                                  | Model + migration + factory |
| `php artisan migrate`                                                | Membuat tabel authors       |
| `php artisan make:migration remove_writer_from_books_table`          | Menghapus kolom writer      |
| `php artisan migrate`                                                | Jalankan penghapusan kolom  |
| `php artisan make:migration create_book_author_table`                | Membuat tabel pivot         |
| `php artisan migrate`                                                | Jalankan migration pivot    |
| `php artisan db:seed`                                                | Mengisi data kategori, penulis, dan buku dengan relasi |
| `php artisan test`                                                   | Menjalankan semua test      |

## 13. Tantangan Lanjutan

Setelah relasi many-to-many berhasil, Anda bisa mencoba:
- Menambahkan atribut pada tabel pivot (misalnya `role` untuk menjelaskan peran penulis dalam buku) dan mengaksesnya dengan `withPivot()`.
- Menggunakan metode `attach()` dan `detach()` untuk menambah atau menghapus penulis individu tanpa mengganti seluruh koleksi.
- Menambahkan filter berdasarkan penulis di halaman index (misalnya menampilkan buku yang ditulis oleh penulis tertentu).
- Membuat relasi many-to-many yang lebih kompleks dengan tiga atau lebih model.

---

Materi terkait: [File `.env`](env.md) · [Composer](composer.md) · [npm dan npx](npm.md) · [CRUD Data Buku](aplikasi-ujicoba-crud.md) · [CRUD dengan Relasi One-to-Many](crud-relasi.md)