# Konvensi: Penamaan, Struktur, dan Code Style

## Penamaan File dan Class

### Modul dan Entity

| Artifact | Format | Contoh |
|---|---|---|
| Folder modul | `PascalCase`, plural | `Products`, `OrderManagement` |
| Eloquent Model | `PascalCase`, singular | `Product`, `OrderItem` |
| Tabel DB | `snake_case`, plural | `products`, `order_items` |
| Migration | `{timestamp}_create_{table}_table.php` | `2024_01_01_000000_create_products_table.php` |

### Application Layer

| Artifact | Format | Contoh |
|---|---|---|
| Action | `{Verb}{Entity}Action` | `CreateProductAction`, `PublishArticleAction` |
| DTO | `{Verb}{Entity}DTO` | `CreateProductDTO`, `UpdateOrderDTO` |
| Repository Interface | `{Entity}RepositoryInterface` | `ProductRepositoryInterface` |
| Repository Impl | `Eloquent{Entity}Repository` | `EloquentProductRepository` |

### Presentation Layer

| Artifact | Format | Contoh |
|---|---|---|
| Controller | `{Entity}Controller` | `ProductController` |
| Form Request (Create) | `Create{Entity}Request` | `CreateProductRequest` |
| Form Request (Update) | `Update{Entity}Request` | `UpdateProductRequest` |
| ViewModel | `{Entity}ViewModel` | `ProductViewModel` |

### Domain Layer

| Artifact | Format | Contoh |
|---|---|---|
| Event | `{Entity}{PastTenseVerb}` | `ProductCreated`, `OrderShipped` |
| Policy | `{Entity}Policy` | `ProductPolicy` |
| Exception | `{Entity}NotFoundException` | `ProductNotFoundException` |
| ServiceProvider | `{Module}ServiceProvider` | `ProductsServiceProvider` |

### Routes dan Views

| Artifact | Format | Contoh |
|---|---|---|
| Route name | `{module}.{action}` | `products.index`, `orders.store` |
| View path (string) | `{module}.{entity}.{action}` | `products.product.index` |
| View file | `{action}.blade.php` | `index.blade.php`, `create.blade.php` |
| Blade component | `{name}.blade.php` | `alert.blade.php`, `card.blade.php` |
| Layout | `{name}.blade.php` di `layouts/` | `app.blade.php`, `auth.blade.php` |
| Partial | `_{name}.blade.php` | `_form.blade.php`, `_navbar.blade.php` |

---

## Namespace Convention

Semua namespace mengikuti PSR-4 dengan root `App\`:

```
app/Modules/Products/Domain/Models/Product.php
→ App\Modules\Products\Domain\Models\Product

app/Modules/Products/Application/Actions/CreateProductAction.php
→ App\Modules\Products\Application\Actions\CreateProductAction

app/Modules/Products/Infrastructure/Repositories/EloquentProductRepository.php
→ App\Modules\Products\Infrastructure\Repositories\EloquentProductRepository

app/Modules/Products/Presentation/Controllers/ProductController.php
→ App\Modules\Products\Presentation\Controllers\ProductController
```

---

## PHP Code Style (PSR-12 + Laravel Conventions)

### Class Declaration

```php
<?php

declare(strict_types=1);

namespace App\Modules\Products\Application\Actions;

// Grouped imports: built-in PHP, then framework, then app-internal
use RuntimeException;
use Illuminate\Support\Facades\DB;
use App\Modules\Products\Application\Contracts\ProductRepositoryInterface;
use App\Modules\Products\Application\DTOs\CreateProductDTO;

final class CreateProductAction
{
    // Constructor property promotion (PHP 8.0+)
    public function __construct(
        private readonly ProductRepositoryInterface $repository,
    ) {}

    public function handle(CreateProductDTO $dto): Product
    {
        // ...
    }
}
```

### Visibility dan `final`

- **Action class**: selalu `final` — tidak untuk di-extend
- **DTO class**: selalu `final` + `readonly` — truly immutable
- **Repository Implementation**: selalu `final`
- **ViewModel**: selalu `final`
- **Eloquent Model**: tidak `final` — Eloquent membutuhkan extension internally
- **Base Controller**: tidak `final` — untuk di-extend

### Type Declarations

```php
// WAJIB: declare(strict_types=1) di setiap file
<?php declare(strict_types=1);

// WAJIB: return type di semua method
public function handle(CreateProductDTO $dto): Product { ... }
public function paginate(int $perPage = 15): LengthAwarePaginator { ... }
public function findById(int $id): ?Product { ... }  // nullable dengan ?

// WAJIB: parameter type hints
public function __construct(
    private readonly ProductRepositoryInterface $repository,
    private readonly EventDispatcherInterface $events,
) {}
```

### Eloquent Model Conventions

```php
final class Product extends Model
{
    // 1. Traits di atas
    use SoftDeletes, HasFactory;

    // 2. Constants
    public const STATUS_ACTIVE   = 'active';
    public const STATUS_INACTIVE = 'inactive';

    // 3. Fillable/Guarded
    protected $fillable = [
        'name', 'price', 'stock', 'category_id', 'description', 'is_active',
    ];

    // 4. Casts (method, bukan property — Laravel 11+)
    protected function casts(): array
    {
        return [
            'price'      => 'decimal:2',
            'is_active'  => 'boolean',
            'metadata'   => 'array',
            'deleted_at' => 'datetime',
        ];
    }

    // 5. Scopes
    public function scopeActive(Builder $query): void
    {
        $query->where('is_active', true);
    }

    // 6. Relationships — WAJIB return type
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function reviews(): HasMany
    {
        return $this->hasMany(Review::class);
    }

    // 7. Business methods (bukan accessor/mutator)
    public function deductStock(int $quantity): void
    {
        if ($this->stock < $quantity) {
            throw new \DomainException("Stok tidak mencukupi.");
        }
        $this->decrement('stock', $quantity);
    }

    // 8. Accessors (Laravel 11 style)
    protected function formattedPrice(): Attribute
    {
        return Attribute::make(
            get: fn () => 'Rp ' . number_format($this->price, 0, ',', '.'),
        );
    }
}
```

### Form Request Conventions

```php
final class CreateProductRequest extends FormRequest
{
    public function authorize(): bool
    {
        // Gunakan Policy untuk otorisasi
        return $this->user()->can('create', Product::class);
    }

    public function rules(): array
    {
        return [
            'name'        => ['required', 'string', 'max:255', 'unique:products,name'],
            'price'       => ['required', 'numeric', 'min:0'],
            'stock'       => ['required', 'integer', 'min:0'],
            'category_id' => ['required', 'exists:categories,id'],
            'description' => ['nullable', 'string', 'max:2000'],
            'is_active'   => ['boolean'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required'        => 'Nama produk wajib diisi.',
            'name.unique'          => 'Nama produk sudah digunakan.',
            'price.required'       => 'Harga produk wajib diisi.',
            'price.numeric'        => 'Harga harus berupa angka.',
            'stock.required'       => 'Stok produk wajib diisi.',
            'category_id.required' => 'Kategori produk wajib dipilih.',
            'category_id.exists'   => 'Kategori tidak ditemukan.',
        ];
    }

    public function attributes(): array
    {
        return [
            'name'        => 'nama produk',
            'price'       => 'harga',
            'stock'       => 'stok',
            'category_id' => 'kategori',
        ];
    }

    // Factory method ada di Form Request (Presentation), bukan di DTO
    // sehingga DTO tetap bebas dari dependency ke Presentation layer
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

Di Controller, panggil via `$request->toDTO()`:
```php
public function store(CreateProductRequest $request): RedirectResponse
{
    $this->createAction->handle($request->toDTO());
    // ...
}
```

---

## Blade Conventions

### Layout Extension

```blade
{{-- resources/views/products/product/index.blade.php --}}
<x-layouts.app title="Daftar Produk">

    <x-slot name="header">
        <h1 class="text-2xl font-semibold">Produk</h1>
    </x-slot>

    {{-- Flash messages ditampilkan otomatis oleh SweetAlert2 toast via layout --}}
    {{-- Tidak perlu <x-alert /> di sini — cukup ->with('success', '...') di controller --}}

    {{-- Content --}}
    <x-card>
        {{-- table atau content --}}
    </x-card>

    {{-- Pagination via component --}}
    <x-pagination :paginator="$products" />

</x-layouts.app>
```

### Output Escaping

```blade
{{-- SELALU gunakan {{ }} untuk output biasa — auto-escaped --}}
{{ $product->name }}
{{ $viewModel->getFormattedPrice() }}

{{-- {!! !!} HANYA untuk HTML yang sudah di-sanitize (terpercaya) --}}
{!! $article->content !!}  {{-- hanya jika content sudah di-sanitize server-side --}}

{{-- JANGAN pernah: --}}
{!! $user->input !!}  {{-- BAHAYA: XSS! --}}
```

### Form Conventions

```blade
<form method="POST" action="{{ route('products.store') }}">
    @csrf

    {{-- Error display per field --}}
    <div>
        <label for="name">Nama Produk</label>
        <input
            type="text"
            id="name"
            name="name"
            value="{{ old('name') }}"
            class="{{ $errors->has('name') ? 'border-red-500' : 'border-gray-300' }}"
        >
        @error('name')
            <p class="text-red-500 text-sm">{{ $message }}</p>
        @enderror
    </div>
</form>

{{-- Update form: method spoofing --}}
<form method="POST" action="{{ route('products.update', $product->id) }}">
    @csrf
    @method('PUT')
    ...
</form>

{{-- Delete button — WAJIB pakai confirmDelete(), BUKAN form inline --}}
<button
    type="button"
    onclick="confirmDelete(
        '{{ route('products.destroy', $product->id) }}',
        '{{ addslashes($product->name) }}'
    )"
    class="text-sm text-red-600 hover:text-red-800">
    Hapus
</button>
```

### Component Usage

```blade
{{-- Flash messages ditampilkan otomatis oleh SweetAlert2 toast —
     cukup ->with('success', '...') di controller, tidak perlu komponen alert di view --}}

{{-- Card component --}}
<x-card title="Daftar Produk" class="mt-4">
    <x-slot name="actions">
        <a href="{{ route('products.create') }}" class="btn btn-primary">Tambah</a>
    </x-slot>
    {{-- card content --}}
</x-card>

{{-- Pagination component --}}
<x-pagination :paginator="$products" />
```

---

## Route Conventions

```php
// app/Modules/Products/routes.php

use App\Modules\Products\Presentation\Controllers\ProductController;
use Illuminate\Support\Facades\Route;

Route::middleware(['web', 'auth'])->group(function () {
    Route::resource('products', ProductController::class)
        ->names('products');  // route names: products.index, products.create, etc.
});
```

Route naming yang digunakan (`Route::resource` otomatis menghasilkan):
- `products.index`   → GET `/products`
- `products.create`  → GET `/products/create`
- `products.store`   → POST `/products`
- `products.show`    → GET `/products/{product}`
- `products.edit`    → GET `/products/{product}/edit`
- `products.update`  → PUT/PATCH `/products/{product}`
- `products.destroy` → DELETE `/products/{product}`

---

## ServiceProvider Convention

```php
// app/Modules/Products/ProductsServiceProvider.php

namespace App\Modules\Products;

use App\Modules\Products\Application\Contracts\ProductRepositoryInterface;
use App\Modules\Products\Infrastructure\Repositories\EloquentProductRepository;
use Illuminate\Support\ServiceProvider;

final class ProductsServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Bind Interface → Implementation (Dependency Inversion)
        $this->app->bind(
            ProductRepositoryInterface::class,
            EloquentProductRepository::class,
        );
    }

    public function boot(): void
    {
        // Load routes dari dalam module
        $this->loadRoutesFrom(__DIR__ . '/routes.php');
    }
}
```

Daftarkan di `bootstrap/providers.php`:
```php
return [
    App\Providers\AppServiceProvider::class,
    App\Modules\Products\ProductsServiceProvider::class,
    // ... modul lain
];
```

---

## Migration Convention

```php
// database/migrations/{timestamp}_create_products_table.php

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name')->unique();
            $table->decimal('price', 12, 2);
            $table->unsignedInteger('stock')->default(0);
            $table->foreignId('category_id')->constrained()->cascadeOnDelete();
            $table->text('description')->nullable();
            $table->boolean('is_active')->default(true);
            // Audit: jika pilih audit timestamps
            $table->string('created_by')->nullable();
            $table->string('updated_by')->nullable();
            $table->timestamps();
            $table->softDeletes();  // jika pilih soft delete

            // Index untuk performa query umum
            $table->index(['is_active', 'created_at']);
            $table->index('category_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
};
```

**Aturan migration:**
- Selalu tambahkan index untuk kolom yang sering di-WHERE, ORDER BY, atau JOIN
- `foreignId()` + `constrained()` untuk foreign key dengan cascade yang sesuai
- `softDeletes()` jika entity memerlukan soft delete
- `timestamps()` selalu ada (kecuali ada alasan kuat tidak perlu)
