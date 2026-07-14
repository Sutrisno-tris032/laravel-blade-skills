# Arsitektur: Clean Architecture + Modular Monolith di Laravel

## Mengapa Clean Architecture untuk Laravel Monolith?

Laravel sangat ekspresif dan bisa langsung berfungsi dengan menaruh logic di Controller. Masalahnya: proyek besar yang semua logic-nya di Controller atau Model menjadi spaghetti code yang sulit di-test dan di-maintain.

**Clean Architecture** memaksa dependency rule yang jelas: layer dalam TIDAK BOLEH tahu tentang layer luar.

**Modular Monolith** mengelompokkan kode per **domain bisnis** (bukan per tipe file), sehingga ketika ada perubahan di fitur `Products`, kita hanya menyentuh folder `app/Modules/Products/`.

**Gabungan keduanya dalam konteks Laravel:**
- Dependency rule ditegakkan (Presentation → Application → Domain)
- Kode diorganisir per modul bisnis (bukan per tipe: Controllers/, Models/, dll.)
- Eloquent tetap digunakan sebagai ORM (pragmatic trade-off: mapper penuh terlalu verbose)
- Repository pattern mengabstraksi Eloquent sehingga Application layer tetap bersih

---

## Struktur Layer

### Domain Layer (`app/Modules/{Module}/Domain/`)

Berisi inti bisnis: Eloquent Model (dengan business rules), Events, Policies, dan Exceptions.

**Trade-off:** Kita menggunakan Eloquent Model (bukan pure PHP object) sebagai domain model. Ini pragmatis — mapper penuh di Laravel menambah kompleksitas yang jarang terbayar. Yang penting: business rules tinggal di dalam Model, bukan di Controller.

```
Domain/
├── Models/
│   └── {Entity}.php          ← Eloquent model dengan business rules
├── Events/
│   ├── {Entity}Created.php
│   ├── {Entity}Updated.php
│   └── {Entity}Deleted.php
├── Policies/
│   └── {Entity}Policy.php
└── Exceptions/
    └── {Entity}NotFoundException.php
```

**Contoh Eloquent Model dengan business rules:**
```php
namespace App\Modules\Products\Domain\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use App\Modules\Products\Domain\Exceptions\ProductNotFoundException;

final class Product extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'name', 'price', 'stock', 'category_id', 'description', 'is_active',
    ];

    protected function casts(): array
    {
        return [
            'price'     => 'decimal:2',
            'is_active' => 'boolean',
        ];
    }

    // Business rule: di dalam model, bukan di controller
    public function deductStock(int $quantity): void
    {
        if ($this->stock < $quantity) {
            throw new \DomainException("Stok tidak mencukupi. Sisa: {$this->stock}");
        }
        $this->decrement('stock', $quantity);
    }

    public function activate(): void
    {
        $this->update(['is_active' => true]);
    }

    // Explicit relationship — return type wajib
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }
}
```

**Aturan Domain:**
- Model WAJIB mendefinisikan `$fillable` (atau `$guarded = []` dengan sadar)
- Model WAJIB mendefinisikan `casts()` untuk type safety
- Relationship WAJIB punya return type hint
- Business rules yang invalid → throw `DomainException` atau custom Exception

---

### Application Layer (`app/Modules/{Module}/Application/`)

Orchestration layer. Mendefinisikan **apa yang dilakukan** sistem (use cases via Actions) dan **kontrak** yang harus dipenuhi Infrastructure (Repository Interfaces).

```
Application/
├── Contracts/
│   └── {Entity}RepositoryInterface.php   ← interface, didefinisikan di sini
├── Actions/
│   ├── Create{Entity}Action.php           ← satu use case = satu Action
│   ├── Update{Entity}Action.php
│   └── Delete{Entity}Action.php
└── DTOs/
    ├── Create{Entity}DTO.php              ← Data Transfer Object, immutable
    └── Update{Entity}DTO.php
```

**Repository Interface (ditetapkan di Application, diimplementasi di Infrastructure):**
```php
namespace App\Modules\Products\Application\Contracts;

use App\Modules\Products\Domain\Models\Product;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Support\LazyCollection;

interface ProductRepositoryInterface
{
    public function findById(int $id): ?Product;
    public function findByIdOrFail(int $id): Product;
    public function paginate(int $perPage = 15, array $with = []): LengthAwarePaginator;
    public function create(array $data): Product;
    public function update(Product $product, array $data): Product;
    public function delete(int $id): void;
    public function search(string $term, int $perPage = 15): LengthAwarePaginator;
    // Untuk large dataset processing
    public function lazyAll(array $with = []): LazyCollection;
}
```

**Action — satu class, satu tanggung jawab:**
```php
namespace App\Modules\Products\Application\Actions;

use App\Modules\Products\Application\Contracts\ProductRepositoryInterface;
use App\Modules\Products\Application\DTOs\CreateProductDTO;
use App\Modules\Products\Domain\Events\ProductCreated;
use App\Modules\Products\Domain\Models\Product;

final class CreateProductAction
{
    public function __construct(
        private readonly ProductRepositoryInterface $repository,
    ) {}

    public function handle(CreateProductDTO $dto): Product
    {
        $product = $this->repository->create([
            'name'        => $dto->name,
            'price'       => $dto->price,
            'stock'       => $dto->stock,
            'category_id' => $dto->categoryId,
            'description' => $dto->description,
            'is_active'   => $dto->isActive,
        ]);

        event(new ProductCreated($product));

        return $product;
    }
}
```

**DTO — immutable, PHP 8.1 readonly:**
```php
namespace App\Modules\Products\Application\DTOs;

// DTO tidak boleh import class dari Presentation layer — murni data object
final readonly class CreateProductDTO
{
    public function __construct(
        public string $name,
        public float  $price,
        public int    $stock,
        public int    $categoryId,
        public string $description,
        public bool   $isActive,
    ) {}
}
```

**Factory method ada di Form Request (Presentation), bukan di DTO:**
```php
namespace App\Modules\Products\Presentation\Requests;

use App\Modules\Products\Application\DTOs\CreateProductDTO;
use Illuminate\Foundation\Http\FormRequest;

final class CreateProductRequest extends FormRequest
{
    // ... authorize(), rules(), messages() ...

    public function toDTO(): CreateProductDTO
    {
        return new CreateProductDTO(
            name:        $this->validated('name'),
            price:       (float) $this->validated('price'),
            stock:       (int) $this->validated('stock'),
            categoryId:  (int) $this->validated('category_id'),
            description: $this->validated('description', ''),
            isActive:    (bool) $this->validated('is_active', true),
        );
    }
}
```

**Aturan Application:**
- Action WAJIB inject Repository Interface — TIDAK BOLEH inject Eloquent Model atau konkret class
- DTO WAJIB `readonly` — tidak boleh dimodifikasi setelah dibuat
- DTO TIDAK BOLEH import class dari Presentation layer (`FormRequest`, `Request`, dll.) — factory method ada di Form Request via `toDTO()`
- Action WAJIB dispatch Event setelah operasi berhasil (untuk decoupling via Listeners)
- Application layer TIDAK BOLEH import `Illuminate\Http\Request` langsung

---

### Infrastructure Layer (`app/Modules/{Module}/Infrastructure/`)

Implementasi konkret dari Repository Interface. Satu-satunya tempat yang boleh langsung berinteraksi dengan Eloquent secara detail.

```
Infrastructure/
└── Repositories/
    └── Eloquent{Entity}Repository.php
```

**Repository Implementation — semua query Eloquent ada di sini:**
```php
namespace App\Modules\Products\Infrastructure\Repositories;

use App\Modules\Products\Application\Contracts\ProductRepositoryInterface;
use App\Modules\Products\Domain\Exceptions\ProductNotFoundException;
use App\Modules\Products\Domain\Models\Product;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Support\LazyCollection;

final class EloquentProductRepository implements ProductRepositoryInterface
{
    public function findById(int $id): ?Product
    {
        return Product::find($id);
    }

    public function findByIdOrFail(int $id): Product
    {
        return Product::findOr($id, fn() => throw new ProductNotFoundException($id));
    }

    public function paginate(int $perPage = 15, array $with = []): LengthAwarePaginator
    {
        // WAJIB eager load relasi yang diketahui — cegah N+1
        return Product::with($with ?: ['category'])
            ->withCount('reviews')
            ->latest()
            ->paginate($perPage);
    }

    public function create(array $data): Product
    {
        return Product::create($data);
    }

    public function update(Product $product, array $data): Product
    {
        $product->update($data);
        return $product->fresh(['category']);  // refresh setelah update
    }

    public function delete(int $id): void
    {
        Product::findOrFail($id)->delete();
    }

    public function search(string $term, int $perPage = 15): LengthAwarePaginator
    {
        return Product::with(['category'])
            ->where(function ($q) use ($term) {
                $q->where('name', 'like', "%{$term}%")
                  ->orWhere('description', 'like', "%{$term}%");
            })
            ->latest()
            ->paginate($perPage);
    }

    // Untuk memproses data besar — memory efficient
    public function lazyAll(array $with = []): LazyCollection
    {
        return Product::with($with)->lazy(200);  // chunk 200 sekaligus
    }
}
```

**Aturan Infrastructure:**
- Repository TIDAK BOLEH berisi business logic (itu milik Domain/Action)
- WAJIB eager load relasi yang diketahui via `with()` — jangan biarkan lazy load terjadi
- Gunakan `fresh()` setelah update agar relasi ter-refresh
- Gunakan `lazy()` untuk dataset besar, bukan `get()` yang load semua ke memori

---

### Presentation Layer (`app/Modules/{Module}/Presentation/`)

Titik masuk HTTP. Hanya bertugas: terima request → delegate ke Action → kembalikan view.

```
Presentation/
├── Controllers/
│   └── {Entity}Controller.php
├── Requests/
│   ├── Create{Entity}Request.php
│   └── Update{Entity}Request.php
└── ViewModels/
    └── {Entity}ViewModel.php
```

**Controller — hanya orchestrate, tidak ada logic:**
```php
namespace App\Modules\Products\Presentation\Controllers;

use App\Http\Controllers\Controller;
use App\Modules\Products\Application\Actions\CreateProductAction;
use App\Modules\Products\Application\Actions\UpdateProductAction;
use App\Modules\Products\Application\Actions\DeleteProductAction;
use App\Modules\Products\Application\Contracts\ProductRepositoryInterface;
use App\Modules\Products\Application\DTOs\CreateProductDTO;
use App\Modules\Products\Application\DTOs\UpdateProductDTO;
use App\Modules\Products\Presentation\Requests\CreateProductRequest;
use App\Modules\Products\Presentation\Requests\UpdateProductRequest;
use App\Modules\Products\Presentation\ViewModels\ProductViewModel;
use Illuminate\View\View;
use Illuminate\Http\RedirectResponse;

final class ProductController extends Controller
{
    public function __construct(
        private readonly ProductRepositoryInterface $repository,
        private readonly CreateProductAction        $createAction,
        private readonly UpdateProductAction        $updateAction,
        private readonly DeleteProductAction        $deleteAction,
    ) {}

    public function index(): View
    {
        $products = $this->repository->paginate(15, ['category']);
        return view('products.product.index', compact('products'));
    }

    public function create(): View
    {
        return view('products.product.create');
    }

    public function store(CreateProductRequest $request): RedirectResponse
    {
        $this->createAction->handle($request->toDTO());

        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil ditambahkan.');
    }

    public function show(int $id): View
    {
        $product   = $this->repository->findByIdOrFail($id);
        $viewModel = new ProductViewModel($product);
        return view('products.product.show', compact('viewModel'));
    }

    public function edit(int $id): View
    {
        $product = $this->repository->findByIdOrFail($id);
        return view('products.product.edit', compact('product'));
    }

    public function update(UpdateProductRequest $request, int $id): RedirectResponse
    {
        $product = $this->repository->findByIdOrFail($id);
        $this->updateAction->handle($product, $request->toDTO());

        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil diperbarui.');
    }

    public function destroy(int $id): RedirectResponse
    {
        $this->deleteAction->handle($id);

        return redirect()->route('products.index')
            ->with('success', 'Produk berhasil dihapus.');
    }
}
```

**ViewModel — format data untuk tampilan:**
```php
namespace App\Modules\Products\Presentation\ViewModels;

use App\Modules\Products\Domain\Models\Product;

final class ProductViewModel
{
    public function __construct(
        private readonly Product $product,
    ) {}

    public function getId(): int                { return $this->product->id; }
    public function getName(): string           { return $this->product->name; }
    public function getDescription(): string    { return $this->product->description ?? ''; }
    public function getCategoryName(): string   { return $this->product->category?->name ?? '-'; }
    public function isActive(): bool            { return $this->product->is_active; }
    public function getStatusLabel(): string    { return $this->product->is_active ? 'Aktif' : 'Nonaktif'; }

    public function getFormattedPrice(): string
    {
        return 'Rp ' . number_format($this->product->price, 0, ',', '.');
    }

    public function getStockStatus(): string
    {
        return match(true) {
            $this->product->stock === 0    => 'Habis',
            $this->product->stock <= 10    => 'Stok Menipis',
            default                        => 'Tersedia',
        };
    }
}
```

---

## Lazy Loading Strategy

### 1. Proteksi N+1 di Development

```php
// app/Providers/AppServiceProvider.php
public function boot(): void
{
    // Throw exception saat lazy loading terjadi di development
    Model::preventLazyLoading(! app()->isProduction());

    // Di production: log warning, jangan crash
    if (app()->isProduction()) {
        Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
            Log::warning('Lazy loading violation', [
                'model'    => get_class($model),
                'relation' => $relation,
            ]);
        });
    }
}
```

### 2. Eager Loading di Repository

```php
// Selalu gunakan with() untuk relasi yang diketahui
public function paginate(int $perPage = 15): LengthAwarePaginator
{
    return Product::with(['category', 'tags'])   // ← explicit eager load
        ->withCount('reviews')
        ->latest()
        ->paginate($perPage);
}
```

### 3. Lazy Collection untuk Dataset Besar

```php
// JANGAN: menggunakan get() untuk semua record
$products = Product::all();  // ← load semua ke memori, bahaya!

// GUNAKAN: lazy() untuk chunked processing
Product::with('category')
    ->where('is_active', true)
    ->lazy(500)  // ambil 500 sekaligus, hemat memori
    ->each(function (Product $product) {
        // proses satu per satu
    });

// GUNAKAN: lazyById() untuk cursor-based (urutan stabil)
Product::lazyById(500, column: 'id')
    ->each(fn (Product $p) => $this->syncToSearchIndex($p));
```

### 4. Cursor Pagination untuk Infinite Scroll

```php
// Lebih efisien dari offset pagination untuk tabel besar
public function cursorPaginate(int $perPage = 20): CursorPaginator
{
    return Product::with('category')
        ->latest()
        ->cursorPaginate($perPage);
}
```

### 5. Conditional Eager Loading

```php
// Load relasi setelah fakta (jika belum di-load)
$product->loadMissing('reviews');

// Load ulang dengan counting
$products->load(['category', 'tags'])->loadCount('reviews');
```

---

## Flow Lengkap: Request ke Response

```
Browser (HTTP Request)
    ↓
routes.php         (Route::resource → ProductController@store)
    ↓
CreateProductRequest::authorize() + rules()  (validasi, 422 jika gagal)
    ↓
ProductController::store()   (delegate ke Action via DTO)
    ↓
CreateProductDTO::fromRequest()   (data immutable dari request)
    ↓
CreateProductAction::handle(dto)   (business logic, inject Repository)
    ↓
ProductRepositoryInterface::create()   (kontrak)
    ↓
EloquentProductRepository::create()   (Eloquent, simpan ke DB)
    ↓
ProductCreated event dispatched   (decoupled side effects)
    ↓
ProductController → redirect()->with('success', ...)
    ↓
Browser (redirect ke index)
```

---

## Keputusan Desain Penting

| Pertanyaan | Jawaban dan Alasan |
|---|---|
| Pakai pure domain objects atau Eloquent? | Eloquent — mapper penuh terlalu verbose untuk most Laravel projects. Masukkan business rules ke dalam model. |
| Pakai Repository pattern? | Ya — memudahkan testing (mock repository), dan memastikan eager loading konsisten. |
| Pakai Service class atau Action? | Action — lebih SRP. Service bisa jadi God Class. |
| ViewModel atau pass Model ke View? | ViewModel — mencegah bisnis logic terselip di Blade, dan formatting terpusat. |
| Lazy loading on atau off? | Off di dev (`preventLazyLoading`), log di production. Semua eager loading via `with()` di repository. |
| Soft delete default? | Tanyakan per modul — tidak semua entity perlu ini. |
| Event per Action? | Ya — memudahkan decoupling side effects (email, log, cache invalidation). |
