# Template Module Slice: CRUD Lengkap

Gunakan template ini saat `new-module`. Ganti:
- `{Module}` → nama modul PascalCase plural (mis. `Products`)
- `{module}` → nama modul snake_case plural (mis. `products`)
- `{Entity}` → nama entity PascalCase singular (mis. `Product`)
- `{entity}` → nama entity snake_case singular (mis. `product`)
- `{entities}` → nama entity snake_case plural (mis. `products`)
- `{FIELDS}` → field yang ditanyakan ke user

---

## Struktur File yang Dibuat

```
app/Modules/{Module}/
├── {Module}ServiceProvider.php
├── routes.php
├── Domain/
│   ├── Models/
│   │   └── {Entity}.php
│   ├── Events/
│   │   ├── {Entity}Created.php
│   │   ├── {Entity}Updated.php
│   │   └── {Entity}Deleted.php
│   ├── Policies/
│   │   └── {Entity}Policy.php
│   └── Exceptions/
│       └── {Entity}NotFoundException.php
├── Application/
│   ├── Contracts/
│   │   └── {Entity}RepositoryInterface.php
│   ├── Actions/
│   │   ├── Create{Entity}Action.php
│   │   ├── Update{Entity}Action.php
│   │   └── Delete{Entity}Action.php
│   └── DTOs/
│       ├── Create{Entity}DTO.php
│       └── Update{Entity}DTO.php
├── Infrastructure/
│   └── Repositories/
│       └── Eloquent{Entity}Repository.php
└── Presentation/
    ├── Controllers/
    │   └── {Entity}Controller.php
    ├── Requests/
    │   ├── Create{Entity}Request.php
    │   └── Update{Entity}Request.php
    └── ViewModels/
        └── {Entity}ViewModel.php

database/migrations/
└── {timestamp}_create_{entities}_table.php

resources/views/{module}/
└── {entity}/
    ├── index.blade.php
    ├── create.blade.php
    ├── edit.blade.php
    ├── show.blade.php
    └── _partials/
        └── form.blade.php
```

---

## File Contents

### Migration: `database/migrations/{timestamp}_create_{entities}_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{entities}', function (Blueprint $table) {
            $table->id();

            // === GANTI: field sesuai jawaban user ===
            // Contoh untuk Products:
            $table->string('name')->unique();
            $table->decimal('price', 12, 2);
            $table->unsignedInteger('stock')->default(0);
            $table->foreignId('category_id')->nullable()->constrained()->nullOnDelete();
            $table->text('description')->nullable();
            $table->boolean('is_active')->default(true);
            // === AKHIR field ===

            // Audit columns (jika user pilih audit timestamps)
            $table->string('created_by')->nullable();
            $table->string('updated_by')->nullable();

            $table->timestamps();
            $table->softDeletes(); // hapus baris ini jika user tidak pilih soft delete

            // Index untuk query yang umum
            $table->index(['is_active', 'created_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{entities}');
    }
};
```

### Domain Model: `app/Modules/{Module}/Domain/Models/{Entity}.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Domain\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;   // hapus jika tidak pilih soft delete

final class {Entity} extends Model
{
    use HasFactory;
    use SoftDeletes;  // hapus jika tidak pilih soft delete

    protected $table = '{entities}';

    protected $fillable = [
        // === GANTI: sesuai field yang dipilih user ===
        'name',
        'price',
        'stock',
        'category_id',
        'description',
        'is_active',
        // === AKHIR fillable ===
        'created_by',  // jika pilih audit
        'updated_by',  // jika pilih audit
    ];

    protected function casts(): array
    {
        return [
            // === SESUAIKAN dengan tipe field ===
            'price'      => 'decimal:2',
            'is_active'  => 'boolean',
            // decimal field lain → 'decimal:2'
            // boolean field    → 'boolean'
            // json field       → 'array'
            // date field       → 'date'
            // datetime field   → 'datetime'
            // === AKHIR casts ===
        ];
    }

    // === Scopes ===
    public function scopeActive(Builder $query): void
    {
        $query->where('is_active', true);
    }

    public function scopeSearch(Builder $query, string $term): void
    {
        $query->where('name', 'like', "%{$term}%");
    }

    // === Relationships (jika ada relasi yang dipilih user) ===
    // Contoh belongsTo:
    // public function category(): BelongsTo
    // {
    //     return $this->belongsTo(Category::class);
    // }

    // === Business Rules ===
    // Taruh validasi business rule di sini, bukan di controller
    // Contoh:
    // public function deductStock(int $qty): void
    // {
    //     if ($this->stock < $qty) {
    //         throw new \DomainException("Stok tidak mencukupi.");
    //     }
    //     $this->decrement('stock', $qty);
    // }
}
```

### Event: `app/Modules/{Module}/Domain/Events/{Entity}Created.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Domain\Events;

use App\Modules\{Module}\Domain\Models\{Entity};
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

final class {Entity}Created
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public readonly {Entity} ${entity},
    ) {}
}
```

### Event: `app/Modules/{Module}/Domain/Events/{Entity}Updated.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Domain\Events;

use App\Modules\{Module}\Domain\Models\{Entity};
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

final class {Entity}Updated
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public readonly {Entity} ${entity},
    ) {}
}
```

### Event: `app/Modules/{Module}/Domain/Events/{Entity}Deleted.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Domain\Events;

use Illuminate\Foundation\Events\Dispatchable;

final class {Entity}Deleted
{
    use Dispatchable;

    public function __construct(
        public readonly int $id,
    ) {}
}
```

### Policy: `app/Modules/{Module}/Domain/Policies/{Entity}Policy.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Domain\Policies;

use App\Models\User;
use App\Modules\{Module}\Domain\Models\{Entity};
use Illuminate\Auth\Access\HandlesAuthorization;

final class {Entity}Policy
{
    use HandlesAuthorization;

    public function viewAny(User $user): bool
    {
        return true;  // atau: $user->hasPermissionTo('view {entities}')
    }

    public function view(User $user, {Entity} ${entity}): bool
    {
        return true;
    }

    public function create(User $user): bool
    {
        return true;  // atau: $user->hasRole('admin')
    }

    public function update(User $user, {Entity} ${entity}): bool
    {
        return true;
    }

    public function delete(User $user, {Entity} ${entity}): bool
    {
        return true;
    }

    public function restore(User $user, {Entity} ${entity}): bool
    {
        return true;
    }

    public function forceDelete(User $user, {Entity} ${entity}): bool
    {
        return true;  // atau: $user->hasRole('admin') — jangan panggil method yang belum ada di model User
    }
}
```

### Exception: `app/Modules/{Module}/Domain/Exceptions/{Entity}NotFoundException.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Domain\Exceptions;

use Illuminate\Database\Eloquent\ModelNotFoundException;

final class {Entity}NotFoundException extends ModelNotFoundException
{
    public function __construct(int|string $id)
    {
        parent::__construct(
            sprintf('{Entity} dengan ID %s tidak ditemukan.', $id)
        );
    }
}
```

### Repository Interface: `app/Modules/{Module}/Application/Contracts/{Entity}RepositoryInterface.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Application\Contracts;

use App\Modules\{Module}\Domain\Models\{Entity};
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Support\LazyCollection;

interface {Entity}RepositoryInterface
{
    public function findById(int $id): ?{Entity};

    public function findByIdOrFail(int $id): {Entity};

    /**
     * Filter + paginate untuk halaman index.
     * $filters: ['search' => '...', 'status' => '', ...field filter lain dari pertanyaan h]
     */
    public function filter(array $filters = [], int $perPage = 15): LengthAwarePaginator;

    public function create(array $data): {Entity};

    public function update({Entity} ${entity}, array $data): {Entity};

    public function delete(int $id): void;

    /** Untuk memproses data besar tanpa memuat semua ke memori (export, batch process) */
    public function lazyAll(array $with = []): LazyCollection;
}
```

### Create DTO: `app/Modules/{Module}/Application/DTOs/Create{Entity}DTO.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Application\DTOs;

// DTO tidak boleh import class dari Presentation layer — murni data object
final readonly class Create{Entity}DTO
{
    public function __construct(
        // === GANTI: sesuai field entity ===
        public string $name,
        public float  $price,
        public int    $stock,
        public string $description,
        public bool   $isActive,
        // === AKHIR fields ===
    ) {}
}
```

### Update DTO: `app/Modules/{Module}/Application/DTOs/Update{Entity}DTO.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Application\DTOs;

final readonly class Update{Entity}DTO
{
    public function __construct(
        // === GANTI: sesuai field entity yang bisa di-update ===
        public string $name,
        public float  $price,
        public int    $stock,
        public string $description,
        public bool   $isActive,
        // === AKHIR fields ===
    ) {}
}
```

### Action Create: `app/Modules/{Module}/Application/Actions/Create{Entity}Action.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Application\Actions;

use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Application\DTOs\Create{Entity}DTO;
use App\Modules\{Module}\Domain\Events\{Entity}Created;
use App\Modules\{Module}\Domain\Models\{Entity};

final class Create{Entity}Action
{
    public function __construct(
        private readonly {Entity}RepositoryInterface $repository,
    ) {}

    public function handle(Create{Entity}DTO $dto): {Entity}
    {
        ${entity} = $this->repository->create([
            // === GANTI: map DTO ke array sesuai fillable ===
            'name'        => $dto->name,
            'price'       => $dto->price,
            'stock'       => $dto->stock,
            'description' => $dto->description,
            'is_active'   => $dto->isActive,
            'created_by'  => auth()->id(),  // hapus jika tidak audit
            // === AKHIR ===
        ]);

        event(new {Entity}Created(${entity}));

        return ${entity};
    }
}
```

### Action Update: `app/Modules/{Module}/Application/Actions/Update{Entity}Action.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Application\Actions;

use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Application\DTOs\Update{Entity}DTO;
use App\Modules\{Module}\Domain\Events\{Entity}Updated;
use App\Modules\{Module}\Domain\Models\{Entity};

final class Update{Entity}Action
{
    public function __construct(
        private readonly {Entity}RepositoryInterface $repository,
    ) {}

    public function handle({Entity} ${entity}, Update{Entity}DTO $dto): {Entity}
    {
        $updated = $this->repository->update(${entity}, [
            // === GANTI: map DTO ke array sesuai fillable ===
            'name'        => $dto->name,
            'price'       => $dto->price,
            'stock'       => $dto->stock,
            'description' => $dto->description,
            'is_active'   => $dto->isActive,
            'updated_by'  => auth()->id(),  // hapus jika tidak audit
            // === AKHIR ===
        ]);

        event(new {Entity}Updated($updated));

        return $updated;
    }
}
```

### Action Delete: `app/Modules/{Module}/Application/Actions/Delete{Entity}Action.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Application\Actions;

use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Domain\Events\{Entity}Deleted;

final class Delete{Entity}Action
{
    public function __construct(
        private readonly {Entity}RepositoryInterface $repository,
    ) {}

    public function handle(int $id): void
    {
        $this->repository->delete($id);
        event(new {Entity}Deleted($id));
    }
}
```

### Repository Implementation: `app/Modules/{Module}/Infrastructure/Repositories/Eloquent{Entity}Repository.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Infrastructure\Repositories;

use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Domain\Exceptions\{Entity}NotFoundException;
use App\Modules\{Module}\Domain\Models\{Entity};
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Support\LazyCollection;

final class Eloquent{Entity}Repository implements {Entity}RepositoryInterface
{
    public function findById(int $id): ?{Entity}
    {
        return {Entity}::find($id);
    }

    public function findByIdOrFail(int $id): {Entity}
    {
        return {Entity}::findOr($id, fn() => throw new {Entity}NotFoundException($id));
    }

    public function filter(array $filters = [], int $perPage = 15): LengthAwarePaginator
    {
        return {Entity}::query()
            // WAJIB eager load relasi — cegah N+1
            // ->with(['category'])
            ->when($filters['search'] ?? null, fn ($q, $v) =>
                $q->where('name', 'like', "%{$v}%"))
            // Filter status — hapus baris ini jika entity tidak punya is_active
            ->when(isset($filters['status']) && $filters['status'] !== '', fn ($q) =>
                $q->where('is_active', (bool) $filters['status']))
            // === Tambah filter sesuai jawaban user (pertanyaan h) ===
            // ->when($filters['category_id'] ?? null, fn ($q, $v) => $q->where('category_id', $v))
            // ->when($filters['date_from'] ?? null, fn ($q, $v) => $q->whereDate('created_at', '>=', $v))
            // ->when($filters['date_to']   ?? null, fn ($q, $v) => $q->whereDate('created_at', '<=', $v))
            // === AKHIR filter ===
            ->latest()
            ->paginate($perPage)
            ->withQueryString();  // pertahankan query string di pagination links
    }

    public function create(array $data): {Entity}
    {
        return {Entity}::create($data);
    }

    public function update({Entity} ${entity}, array $data): {Entity}
    {
        ${entity}->update($data);
        return ${entity}->fresh();  // refresh agar data ter-update
    }

    public function delete(int $id): void
    {
        {Entity}::findOrFail($id)->delete();
    }

    public function lazyAll(array $with = []): LazyCollection
    {
        return {Entity}::with($with)->lazy(200);  // chunk 200 sekaligus, hemat memori
    }
}
```

### Controller: `app/Modules/{Module}/Presentation/Controllers/{Entity}Controller.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Presentation\Controllers;

use App\Http\Controllers\Controller;
use App\Modules\{Module}\Application\Actions\Create{Entity}Action;
use App\Modules\{Module}\Application\Actions\Delete{Entity}Action;
use App\Modules\{Module}\Application\Actions\Update{Entity}Action;
use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Presentation\Requests\Create{Entity}Request;
use App\Modules\{Module}\Presentation\Requests\Update{Entity}Request;
use App\Modules\{Module}\Presentation\ViewModels\{Entity}ViewModel;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

final class {Entity}Controller extends Controller
{
    public function __construct(
        private readonly {Entity}RepositoryInterface $repository,
        private readonly Create{Entity}Action        $createAction,
        private readonly Update{Entity}Action        $updateAction,
        private readonly Delete{Entity}Action        $deleteAction,
    ) {}

    public function index(Request $request): View
    {
        // Sesuaikan key filter dengan jawaban user (pertanyaan h)
        $filters    = $request->only(['search', 'status']);
        ${entities} = $this->repository->filter($filters, 15);

        return view('{module}.{entity}.index', compact('{entities}', 'filters'));
    }

    public function create(): View
    {
        // === Jika ada relasi belongsTo (pertanyaan g): load options untuk <x-form.select> ===
        // Contoh: $categories = $this->categoryRepository->options();  — lihat §Pola Relasi di Form
        // return view('{module}.{entity}.create', compact('categories'));
        // ===
        return view('{module}.{entity}.create');
    }

    public function store(Create{Entity}Request $request): RedirectResponse
    {
        $this->createAction->handle($request->toDTO());

        return redirect()->route('{module}.index')
            ->with('success', '{Entity} berhasil ditambahkan.');
    }

    public function show(int $id): View
    {
        ${entity}    = $this->repository->findByIdOrFail($id);
        $viewModel   = new {Entity}ViewModel(${entity});
        return view('{module}.{entity}.show', compact('viewModel'));
    }

    public function edit(int $id): View
    {
        ${entity} = $this->repository->findByIdOrFail($id);
        // === Jika ada relasi belongsTo: tambahkan options sama seperti create() ===
        // return view('{module}.{entity}.edit', compact('{entity}', 'categories'));
        // ===
        return view('{module}.{entity}.edit', compact('{entity}'));
    }

    public function update(Update{Entity}Request $request, int $id): RedirectResponse
    {
        ${entity} = $this->repository->findByIdOrFail($id);
        $this->updateAction->handle(${entity}, $request->toDTO());

        return redirect()->route('{module}.index')
            ->with('success', '{Entity} berhasil diperbarui.');
    }

    public function destroy(int $id): RedirectResponse
    {
        $this->deleteAction->handle($id);

        return redirect()->route('{module}.index')
            ->with('success', '{Entity} berhasil dihapus.');
    }
}
```

### Form Request Create: `app/Modules/{Module}/Presentation/Requests/Create{Entity}Request.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Presentation\Requests;

use App\Modules\{Module}\Application\DTOs\Create{Entity}DTO;
use App\Modules\{Module}\Domain\Models\{Entity};
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

final class Create{Entity}Request extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('create', {Entity}::class);
    }

    public function rules(): array
    {
        return [
            // === GANTI: sesuai field entity ===
            // Jika entity pakai Soft Delete (pertanyaan e): tambahkan ->whereNull('deleted_at')
            // agar nama yang sudah di-soft-delete tidak memblokir pembuatan record baru dengan nama sama.
            'name'        => ['required', 'string', 'max:255', Rule::unique('{entities}', 'name')],
            'price'       => ['required', 'numeric', 'min:0'],
            'stock'       => ['required', 'integer', 'min:0'],
            'description' => ['nullable', 'string', 'max:5000'],
            'is_active'   => ['boolean'],
            // === AKHIR rules ===
        ];
    }

    public function messages(): array
    {
        return [
            // === GANTI: sesuai field dan bahasa yang diinginkan ===
            'name.required'  => 'Nama wajib diisi.',
            'name.unique'    => 'Nama sudah digunakan.',
            'price.required' => 'Harga wajib diisi.',
            'price.numeric'  => 'Harga harus berupa angka.',
            'stock.required' => 'Stok wajib diisi.',
            // === AKHIR messages ===
        ];
    }

    // Factory method — agar DTO tidak perlu import class Presentation
    public function toDTO(): Create{Entity}DTO
    {
        return new Create{Entity}DTO(
            // === GANTI: sesuai field entity ===
            name:        $this->validated('name'),
            price:       (float) $this->validated('price'),
            stock:       (int) $this->validated('stock'),
            description: $this->validated('description', ''),
            isActive:    (bool) $this->validated('is_active', true),
            // === AKHIR ===
        );
    }
}
```

### Form Request Update: `app/Modules/{Module}/Presentation/Requests/Update{Entity}Request.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Presentation\Requests;

use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Application\DTOs\Update{Entity}DTO;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

final class Update{Entity}Request extends FormRequest
{
    public function __construct(
        private readonly {Entity}RepositoryInterface $repository,
    ) {
        parent::__construct();
    }

    public function authorize(): bool
    {
        $id = $this->route('{entity}');
        ${entity} = $this->repository->findByIdOrFail($id);
        return $this->user()->can('update', ${entity});
    }

    public function rules(): array
    {
        $id = $this->route('{entity}');
        return [
            // === GANTI: sesuai field entity — biasanya sama dengan Create, kecuali unique pakai ignore ===
            // Jika entity pakai Soft Delete (pertanyaan e): tambahkan ->whereNull('deleted_at')
            'name'        => ['required', 'string', 'max:255', Rule::unique('{entities}', 'name')->ignore($id)],
            'price'       => ['required', 'numeric', 'min:0'],
            'stock'       => ['required', 'integer', 'min:0'],
            'description' => ['nullable', 'string', 'max:5000'],
            'is_active'   => ['boolean'],
            // === AKHIR rules ===
        ];
    }

    public function messages(): array
    {
        return [
            // === GANTI: sesuai field ===
            'name.required'  => 'Nama wajib diisi.',
            'name.unique'    => 'Nama sudah digunakan.',
            'price.required' => 'Harga wajib diisi.',
            'stock.required' => 'Stok wajib diisi.',
            // === AKHIR messages ===
        ];
    }

    public function toDTO(): Update{Entity}DTO
    {
        return new Update{Entity}DTO(
            // === GANTI: sesuai field entity ===
            name:        $this->validated('name'),
            price:       (float) $this->validated('price'),
            stock:       (int) $this->validated('stock'),
            description: $this->validated('description', ''),
            isActive:    (bool) $this->validated('is_active', true),
            // === AKHIR ===
        );
    }
}
```

### ViewModel: `app/Modules/{Module}/Presentation/ViewModels/{Entity}ViewModel.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module}\Presentation\ViewModels;

use App\Modules\{Module}\Domain\Models\{Entity};

final class {Entity}ViewModel
{
    public function __construct(
        private readonly {Entity} ${entity},
    ) {}

    public function getId(): int    { return $this->{entity}->id; }
    public function getName(): string { return $this->{entity}->name; }

    // === GANTI: tambahkan getter yang relevan sesuai field ===
    public function getFormattedPrice(): string
    {
        return 'Rp ' . number_format($this->{entity}->price, 0, ',', '.');
    }

    public function getStockLabel(): string
    {
        return match(true) {
            $this->{entity}->stock === 0  => 'Habis',
            $this->{entity}->stock <= 5   => 'Stok Menipis',
            default                       => 'Tersedia',
        };
    }

    public function isActive(): bool    { return $this->{entity}->is_active; }
    public function getStatusLabel(): string
    {
        return $this->{entity}->is_active ? 'Aktif' : 'Nonaktif';
    }

    public function getCreatedAt(): string
    {
        return $this->{entity}->created_at?->translatedFormat('d F Y, H:i') ?? '-';
    }
    // === AKHIR getters ===

    /** Kembalikan model asli untuk akses di luar getter (avoid direct usage jika bisa) */
    public function toModel(): {Entity} { return $this->{entity}; }
}
```

### ServiceProvider: `app/Modules/{Module}/{Module}ServiceProvider.php`

```php
<?php

declare(strict_types=1);

namespace App\Modules\{Module};

use App\Modules\{Module}\Application\Contracts\{Entity}RepositoryInterface;
use App\Modules\{Module}\Domain\Models\{Entity};
use App\Modules\{Module}\Domain\Policies\{Entity}Policy;
use App\Modules\{Module}\Infrastructure\Repositories\Eloquent{Entity}Repository;
use Illuminate\Support\Facades\Gate;
use Illuminate\Support\ServiceProvider;

final class {Module}ServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->bind(
            {Entity}RepositoryInterface::class,
            Eloquent{Entity}Repository::class,
        );
    }

    public function boot(): void
    {
        $this->loadRoutesFrom(__DIR__ . '/routes.php');

        // Register Policy
        Gate::policy({Entity}::class, {Entity}Policy::class);
    }
}
```

### Routes: `app/Modules/{Module}/routes.php`

```php
<?php

use App\Modules\{Module}\Presentation\Controllers\{Entity}Controller;
use Illuminate\Support\Facades\Route;

Route::middleware(['web', 'auth'])->group(function () {
    Route::resource('{module}', {Entity}Controller::class)
        ->names('{module}');
});
```

---

## Blade Views

### `resources/views/{module}/{entity}/index.blade.php`

```blade
<x-layouts.app title="Daftar {Entity}">

    <x-slot name="header">
        <h1 class="text-2xl font-semibold text-gray-900">Daftar {Entity}</h1>
    </x-slot>

    <x-card>
        <x-slot name="actions">
            {{-- Filter form — tambah/hapus field sesuai pertanyaan h --}}
            <form method="GET" action="{{ route('{module}.index') }}" class="flex flex-wrap items-center gap-2">

                {{-- Search keyword --}}
                <input
                    type="text"
                    name="search"
                    value="{{ $filters['search'] ?? '' }}"
                    placeholder="Cari {entity}..."
                    class="block rounded-lg border-gray-300 shadow-sm text-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
                >

                {{-- Filter status — hapus jika tidak dipilih di pertanyaan h --}}
                <select name="status"
                        class="block rounded-lg border-gray-300 shadow-sm text-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500">
                    <option value="">Semua Status</option>
                    <option value="1" {{ ($filters['status'] ?? '') === '1' ? 'selected' : '' }}>Aktif</option>
                    <option value="0" {{ ($filters['status'] ?? '') === '0' ? 'selected' : '' }}>Nonaktif</option>
                </select>

                {{-- === Tambah filter lain di sini (pertanyaan h) ===
                <select name="category_id" class="...">
                    <option value="">Semua Kategori</option>
                    ...
                </select>
                === AKHIR filter tambahan === --}}

                <x-button type="submit" variant="secondary" size="sm">Filter</x-button>

                @if(array_filter($filters ?? []))
                    <x-button href="{{ route('{module}.index') }}" variant="ghost" size="sm">Reset</x-button>
                @endif
            </form>

            {{-- Export button — tambahkan jika pilih Ya di pertanyaan i --}}
            {{-- <x-button href="{{ route('{module}.export') }}" variant="secondary" size="sm">
                <x-heroicon-o-arrow-down-tray class="w-4 h-4" />
                Export
            </x-button> --}}

            <x-button href="{{ route('{module}.create') }}">+ Tambah {Entity}</x-button>
        </x-slot>

        @if(${entities}->isEmpty())
            <x-empty-state
                title="Belum ada {entity}"
                description="Mulai dengan menambahkan {entity} pertama.">
                <x-slot name="action">
                    <x-button href="{{ route('{module}.create') }}">+ Tambah {Entity}</x-button>
                </x-slot>
            </x-empty-state>
        @else
            <div class="overflow-x-auto">
                <table class="min-w-full divide-y divide-gray-200 text-sm">
                    <thead class="bg-gray-50">
                        <tr>
                            <th class="px-4 py-3 text-left font-medium text-gray-500">#</th>
                            {{-- === GANTI: kolom sesuai field entity === --}}
                            <th class="px-4 py-3 text-left font-medium text-gray-500">Nama</th>
                            <th class="px-4 py-3 text-left font-medium text-gray-500">Harga</th>
                            <th class="px-4 py-3 text-left font-medium text-gray-500">Stok</th>
                            <th class="px-4 py-3 text-left font-medium text-gray-500">Status</th>
                            {{-- === AKHIR kolom === --}}
                            <th class="px-4 py-3 text-right font-medium text-gray-500">Aksi</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-gray-200">
                        @foreach(${entities} as $index => ${entity})
                            <tr class="hover:bg-gray-50">
                                <td class="px-4 py-3 text-gray-500">
                                    {{ (${entities}->currentPage() - 1) * ${entities}->perPage() + $index + 1 }}
                                </td>
                                {{-- === GANTI: cell sesuai field === --}}
                                <td class="px-4 py-3 font-medium text-gray-900">{{ ${entity}->name }}</td>
                                <td class="px-4 py-3">Rp {{ number_format(${entity}->price, 0, ',', '.') }}</td>
                                <td class="px-4 py-3">{{ ${entity}->stock }}</td>
                                <td class="px-4 py-3">
                                    <x-badge :type="${entity}->is_active ? 'success' : 'danger'" :dot="true">
                                        {{ ${entity}->is_active ? 'Aktif' : 'Nonaktif' }}
                                    </x-badge>
                                </td>
                                {{-- === AKHIR cell === --}}
                                <td class="px-4 py-3 text-right">
                                    <div class="flex justify-end gap-2">
                                        <a href="{{ route('{module}.show', ${entity}->id) }}"
                                           class="text-blue-600 hover:text-blue-800 text-xs">Detail</a>
                                        <a href="{{ route('{module}.edit', ${entity}->id) }}"
                                           class="text-yellow-600 hover:text-yellow-800 text-xs">Edit</a>
                                        {{-- Hapus: gunakan SweetAlert2 confirmDelete() --}}
                                        <button
                                            type="button"
                                            onclick="confirmDelete(
                                                '{{ route('{module}.destroy', ${entity}->id) }}',
                                                '{{ addslashes(${entity}->name) }}'
                                            )"
                                            class="text-xs font-medium text-red-600 hover:text-red-800 transition-colors">
                                            Hapus
                                        </button>
                                    </div>
                                </td>
                            </tr>
                        @endforeach
                    </tbody>
                </table>
            </div>

            <x-pagination :paginator="${entities}" />
        @endif
    </x-card>

</x-layouts.app>
```

### `resources/views/{module}/{entity}/_partials/form.blade.php`

```blade
{{-- Form partial — di-@include (bukan komponen) dari create.blade.php dan edit.blade.php --}}
{{-- $entity WAJIB selalu di-pass dari pemanggil via @include(..., ['entity' => ...]) — @props tidak berlaku untuk @include --}}

{{-- === GANTI: field sesuai entity — gunakan komponen <x-form.*> === --}}
<div class="grid grid-cols-1 gap-6 sm:grid-cols-2">

    <div class="sm:col-span-2">
        <x-form.input
            name="name"
            label="Nama"
            :required="true"
            value="{{ old('name', $entity?->name) }}"
        />
    </div>

    {{-- Contoh field decimal dengan prefix currency --}}
    <x-form.input
        name="price"
        label="Harga"
        type="number"
        prefix="Rp"
        :required="true"
        value="{{ old('price', $entity?->price) }}"
        min="0"
        step="0.01"
    />

    {{-- Contoh field integer --}}
    <x-form.input
        name="stock"
        label="Stok"
        type="number"
        :required="true"
        value="{{ old('stock', $entity?->stock ?? 0) }}"
        min="0"
    />

    <div class="sm:col-span-2">
        <x-form.textarea
            name="description"
            label="Deskripsi"
            :rows="4"
            value="{{ old('description', $entity?->description) }}"
        />
    </div>

    <div class="sm:col-span-2">
        <x-form.checkbox
            name="is_active"
            label="Aktif"
            :checked="old('is_active', $entity?->is_active ?? true)"
        />
    </div>

    {{-- === Jika ada relasi belongsTo (pertanyaan g): uncomment dan sesuaikan === --}}
    {{--
    <x-form.select
        name="category_id"
        label="Kategori"
        :required="true"
        :options="$categories->pluck('name', 'id')"
        :value="old('category_id', $entity?->category_id)"
    />
    --}}

    {{-- === Jika ada file/gambar (pertanyaan j): uncomment dan sesuaikan === --}}
    {{-- Ingat: tambahkan enctype="multipart/form-data" di tag <form> di create/edit view --}}
    {{--
    <div class="sm:col-span-2">
        <x-form.file
            name="photo"
            label="Foto"
            accept="image/*"
            :current="$entity?->photo_path"
            hint="Format: JPG, PNG, WebP. Maks 2MB."
        />
    </div>
    --}}

</div>
{{-- === AKHIR field === --}}
```

### `resources/views/{module}/{entity}/create.blade.php`

```blade
<x-layouts.app title="Tambah {Entity}">

    <x-slot name="header">
        <div class="flex items-center gap-2">
            <a href="{{ route('{module}.index') }}" class="text-gray-500 hover:text-gray-700">← Kembali</a>
            <span class="text-gray-400">/</span>
            <h1 class="text-2xl font-semibold text-gray-900">Tambah {Entity}</h1>
        </div>
    </x-slot>

    <x-card>
        <form method="POST" action="{{ route('{module}.store') }}">
            @csrf

            @include('{module}.{entity}._partials.form', ['entity' => null])

            <div class="mt-8 pt-6 border-t border-gray-200 flex justify-end gap-3">
                <x-button variant="secondary" href="{{ route('{module}.index') }}">Batal</x-button>
                <x-button type="submit">Simpan</x-button>
            </div>
        </form>
    </x-card>

</x-layouts.app>
```

### `resources/views/{module}/{entity}/edit.blade.php`

```blade
<x-layouts.app title="Edit {Entity}">

    <x-slot name="header">
        <div class="flex items-center gap-2">
            <a href="{{ route('{module}.index') }}" class="text-gray-500 hover:text-gray-700">← Kembali</a>
            <span class="text-gray-400">/</span>
            <h1 class="text-2xl font-semibold text-gray-900">Edit {Entity}</h1>
        </div>
    </x-slot>

    <x-card>
        <form method="POST" action="{{ route('{module}.update', ${entity}->id) }}">
            @csrf
            @method('PUT')

            @include('{module}.{entity}._partials.form', ['entity' => ${entity}])

            <div class="mt-8 pt-6 border-t border-gray-200 flex justify-end gap-3">
                <x-button variant="secondary" href="{{ route('{module}.index') }}">Batal</x-button>
                <x-button type="submit">Simpan Perubahan</x-button>
            </div>
        </form>
    </x-card>

</x-layouts.app>
```

### `resources/views/{module}/{entity}/show.blade.php`

```blade
<x-layouts.app title="Detail {Entity}">

    <x-slot name="header">
        <div class="flex items-center justify-between">
            <div class="flex items-center gap-2">
                <a href="{{ route('{module}.index') }}" class="text-gray-500 hover:text-gray-700">← Kembali</a>
                <span class="text-gray-400">/</span>
                <h1 class="text-2xl font-semibold text-gray-900">{{ $viewModel->getName() }}</h1>
            </div>
            <div class="flex gap-2">
                <a href="{{ route('{module}.edit', $viewModel->getId()) }}"
                   class="px-3 py-2 bg-yellow-500 text-white text-sm rounded-md hover:bg-yellow-600">
                    Edit
                </a>
                {{-- Hapus: SweetAlert2 confirmDelete() — tidak ada form inline --}}
                <button
                    type="button"
                    onclick="confirmDelete(
                        '{{ route('{module}.destroy', $viewModel->getId()) }}',
                        '{{ addslashes($viewModel->getName()) }}'
                    )"
                    class="px-3 py-2 bg-red-600 text-white text-sm rounded-lg hover:bg-red-700 transition-colors">
                    Hapus
                </button>
            </div>
        </div>
    </x-slot>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <x-card title="Informasi {Entity}">
            <dl class="space-y-3 text-sm">
                {{-- === GANTI: sesuai field via ViewModel === --}}
                <div class="flex justify-between">
                    <dt class="text-gray-500">Nama</dt>
                    <dd class="font-medium text-gray-900">{{ $viewModel->getName() }}</dd>
                </div>
                <div class="flex justify-between">
                    <dt class="text-gray-500">Harga</dt>
                    <dd class="font-medium text-gray-900">{{ $viewModel->getFormattedPrice() }}</dd>
                </div>
                <div class="flex justify-between">
                    <dt class="text-gray-500">Stok</dt>
                    <dd class="font-medium">{{ $viewModel->getStockLabel() }}</dd>
                </div>
                <div class="flex justify-between">
                    <dt class="text-gray-500">Status</dt>
                    <dd class="font-medium {{ $viewModel->isActive() ? 'text-green-600' : 'text-gray-400' }}">
                        {{ $viewModel->getStatusLabel() }}
                    </dd>
                </div>
                <div class="flex justify-between">
                    <dt class="text-gray-500">Dibuat</dt>
                    <dd class="text-gray-900">{{ $viewModel->getCreatedAt() }}</dd>
                </div>
                {{-- === AKHIR detail === --}}
            </dl>
        </x-card>
    </div>

</x-layouts.app>
```

---

---

## Pola Fitur Tambahan

Bagian ini digunakan berdasarkan jawaban user saat `new-module`. Terapkan pola yang relevan saja.

---

### Pola Relasi di Form (pertanyaan g)

**Dependency Rule tetap berlaku lintas modul** — Controller TIDAK BOLEH query Eloquent model modul lain secara langsung. Tambahkan method khusus (mis. `options()`) di Repository Interface modul terkait (mis. `CategoryRepositoryInterface`), lalu inject interface itu — bukan Model-nya — via constructor:

```php
// Tambahkan ke App\Modules\Categories\Application\Contracts\CategoryRepositoryInterface
public function options(): \Illuminate\Support\Collection;
```

```php
// Implementasi di App\Modules\Categories\Infrastructure\Repositories\EloquentCategoryRepository
public function options(): \Illuminate\Support\Collection
{
    return Category::query()
        ->select('id', 'name')
        ->where('is_active', true)
        ->orderBy('name')
        ->pluck('name', 'id');
}
```

**Controller `create()` dan `edit()`** — inject `CategoryRepositoryInterface` via constructor (tambahkan ke daftar constructor property yang sudah ada), lalu panggil `options()`:

```php
public function __construct(
    private readonly {Entity}RepositoryInterface $repository,
    private readonly CategoryRepositoryInterface  $categoryRepository, // ← tambahan untuk relasi
    // ...Actions lain seperti biasa
) {}

public function create(): View
{
    $categories = $this->categoryRepository->options();

    return view('{module}.{entity}.create', compact('categories'));
}

public function edit(int $id): View
{
    ${entity}   = $this->repository->findByIdOrFail($id);
    $categories = $this->categoryRepository->options();

    return view('{module}.{entity}.edit', compact('{entity}', 'categories'));
}
```

**Form partial** — ganti `<x-form.select>` yang di-comment:

```blade
<x-form.select
    name="category_id"
    label="Kategori"
    :required="true"
    :options="$categories->pluck('name', 'id')"
    :value="old('category_id', $entity?->category_id)"
/>
```

**DTO** — tambahkan field relasi:

```php
public int $categoryId,
```

**Form Request `toDTO()`** — tambahkan field relasi di dalam method `toDTO()`:

```php
categoryId: (int) $this->validated('category_id'),
```

**Form Request rules** — validasi foreign key:

```php
'category_id' => ['required', 'exists:categories,id'],
```

---

### Pola Filter Multi-Kolom (pertanyaan h)

Filter lebih kompleks yang sudah ada di `filter()` di Repository. Untuk filter **date range**, tambahkan ke Repository:

```php
->when($filters['date_from'] ?? null, fn ($q, $v) =>
    $q->whereDate('created_at', '>=', $v))
->when($filters['date_to'] ?? null, fn ($q, $v) =>
    $q->whereDate('created_at', '<=', $v))
```

Tambah input date di index view (dalam `<form method="GET">`):

```blade
<input type="date" name="date_from"
       value="{{ $filters['date_from'] ?? '' }}"
       class="block rounded-lg border-gray-300 shadow-sm text-sm">
<input type="date" name="date_to"
       value="{{ $filters['date_to'] ?? '' }}"
       class="block rounded-lg border-gray-300 shadow-sm text-sm">
```

Update `$filters = $request->only([...])` di Controller untuk include key baru:

```php
$filters = $request->only(['search', 'status', 'date_from', 'date_to']);
```

---

### Pola Export Excel/CSV (pertanyaan i = Ya)

**Tambah package:**
```bash
composer require rap2hpoutre/fast-excel
```

**Controller** — tambahkan method `export()` dan import:

```php
use Rap2hpoutre\FastExcel\FastExcel;
use Symfony\Component\HttpFoundation\StreamedResponse;

public function export(): StreamedResponse
{
    return (new FastExcel($this->repository->lazyAll()))
        ->download('{entities}-' . now()->format('Y-m-d') . '.xlsx', function ({Entity} ${entity}) {
            return [
                'ID'       => ${entity}->id,
                // === Sesuaikan kolom dengan field entity ===
                'Nama'     => ${entity}->name,
                'Status'   => ${entity}->is_active ? 'Aktif' : 'Nonaktif',
                'Dibuat'   => ${entity}->created_at?->format('d/m/Y'),
                // ===
            ];
        });
}
```

**Route** — tambahkan di `routes.php` (sebelum `Route::resource`):

```php
Route::middleware(['web', 'auth'])->group(function () {
    // Export harus sebelum resource agar tidak ditangkap sebagai {id}
    Route::get('{module}/export', [{Entity}Controller::class, 'export'])
        ->name('{module}.export');

    Route::resource('{module}', {Entity}Controller::class)
        ->names('{module}');
});
```

**View** — uncomment tombol export di index:

```blade
<x-button href="{{ route('{module}.export') }}" variant="secondary" size="sm">
    <x-heroicon-o-arrow-down-tray class="w-4 h-4" />
    Export
</x-button>
```

---

### Pola File Upload (pertanyaan j)

**Migration** — tambahkan kolom file:

```php
$table->string('photo_path')->nullable();   // untuk gambar
// atau
$table->string('document_path')->nullable(); // untuk dokumen
```

**Model** — tambahkan ke `$fillable`:

```php
'photo_path',
```

**DTO** — tambahkan field:

```php
public ?string $photoPath,
```

**Form Request `toDTO()`** — `photoPath` tidak di-pass dari request, diisi di Action setelah file diupload:

```php
public function toDTO(): Create{Entity}DTO
{
    return new Create{Entity}DTO(
        name:      $this->validated('name'),
        // ... field lain sesuai entity ...
        photoPath: null,  // akan diisi di Action setelah upload
    );
}
```

**Action Create** — handle upload sebelum simpan ke repository:

```php
use Illuminate\Support\Facades\Storage;

public function handle(Create{Entity}DTO $dto, ?\Illuminate\Http\UploadedFile $photo = null): {Entity}
{
    $data = [
        'name'       => $dto->name,
        // ... field lain
        'created_by' => auth()->id(),
    ];

    // Upload file jika ada
    if ($photo !== null) {
        $data['photo_path'] = $photo->store('{entities}/photos', 'public');
    }

    ${entity} = $this->repository->create($data);
    event(new {Entity}Created(${entity}));
    return ${entity};
}
```

**Action Update** — handle upload + hapus file lama:

```php
public function handle({Entity} ${entity}, Update{Entity}DTO $dto, ?\Illuminate\Http\UploadedFile $photo = null): {Entity}
{
    $data = [ /* ... field lain ... */ ];

    if ($photo !== null) {
        // Hapus file lama sebelum upload baru
        if (${entity}->photo_path) {
            Storage::disk('public')->delete(${entity}->photo_path);
        }
        $data['photo_path'] = $photo->store('{entities}/photos', 'public');
    }

    $updated = $this->repository->update(${entity}, $data);
    event(new {Entity}Updated($updated));
    return $updated;
}
```

**Controller `store()` dan `update()`** — teruskan file ke Action:

```php
public function store(Create{Entity}Request $request): RedirectResponse
{
    $this->createAction->handle(
        $request->toDTO(),
        $request->file('photo'),  // null jika tidak ada
    );
    return redirect()->route('{module}.index')->with('success', '{Entity} berhasil ditambahkan.');
}

public function update(Update{Entity}Request $request, int $id): RedirectResponse
{
    ${entity} = $this->repository->findByIdOrFail($id);
    $this->updateAction->handle(
        ${entity},
        $request->toDTO(),
        $request->file('photo'),
    );
    return redirect()->route('{module}.index')->with('success', '{Entity} berhasil diperbarui.');
}
```

**Form Request** — validasi file:

```php
'photo' => ['nullable', 'image', 'max:2048'],       // gambar, maks 2MB
// atau
'document' => ['nullable', 'file', 'mimes:pdf,doc,docx', 'max:5120'],  // dokumen, maks 5MB
```

**Form partial** — uncomment `<x-form.file>` dan tambahkan `enctype`:

Di `create.blade.php` dan `edit.blade.php`, tambahkan `enctype` ke form:

```blade
<form method="POST" action="{{ route('{module}.store') }}" enctype="multipart/form-data">
```

**Action Delete** — hapus file saat entity dihapus:

```php
public function handle(int $id): void
{
    ${entity} = $this->repository->findById($id);
    if (${entity}?->photo_path) {
        Storage::disk('public')->delete(${entity}->photo_path);
    }
    $this->repository->delete($id);
    event(new {Entity}Deleted($id));
}
```

**Tampilkan gambar di show view:**

```blade
@if ($viewModel->toModel()->photo_path)
    <img src="{{ Storage::url($viewModel->toModel()->photo_path) }}"
         alt="{{ $viewModel->getName() }}"
         class="w-full max-w-sm rounded-xl border border-gray-200 object-cover">
@endif
```

---

### Pola Role & Permission — RBAC (pertanyaan k = 2)

**Install package:**
```bash
composer require spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

**Tambahkan trait ke `User` model** (`app/Models/User.php`):

```php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
}
```

**Policy** — ganti semua `return true` dengan permission check:

```php
public function viewAny(User $user): bool
{
    return $user->hasPermissionTo('view {entities}');
}

public function create(User $user): bool
{
    return $user->hasPermissionTo('create {entities}');
}

public function update(User $user, {Entity} ${entity}): bool
{
    return $user->hasPermissionTo('edit {entities}');
}

public function delete(User $user, {Entity} ${entity}): bool
{
    return $user->hasPermissionTo('delete {entities}');
}
```

**Form Request** — `authorize()` sudah benar (delegasi ke Policy), tidak perlu ubah:

```php
public function authorize(): bool
{
    return $this->user()->can('create', {Entity}::class);  // memanggil Policy::create()
}
```

**Seed permissions** di `DatabaseSeeder.php`:

```php
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

// Buat permissions
$permissions = [
    'view {entities}', 'create {entities}', 'edit {entities}', 'delete {entities}',
];
foreach ($permissions as $perm) {
    Permission::firstOrCreate(['name' => $perm]);
}

// Buat role admin dengan semua permission
$admin = Role::firstOrCreate(['name' => 'admin']);
$admin->syncPermissions($permissions);
```

---

## Daftarkan Modul ke `bootstrap/providers.php`

Setelah semua file dibuat, tambahkan ServiceProvider modul ke `bootstrap/providers.php`:

```php
return [
    App\Providers\AppServiceProvider::class,
    // Tambahkan baris ini:
    App\Modules\{Module}\{Module}ServiceProvider::class,
];
```

---

## Checklist Final Setelah `new-module`

- [ ] Migration dibuat di `database/migrations/`
- [ ] Eloquent Model punya `$fillable` dan `casts()`
- [ ] Repository Interface ada di `Application/Contracts/`
- [ ] Repository Implementation di `Infrastructure/Repositories/`
- [ ] Actions (Create, Update, Delete) ada di `Application/Actions/`
- [ ] DTOs (Create, Update) ada di `Application/DTOs/`
- [ ] Controller inject via constructor (bukan instantiate langsung)
- [ ] Form Requests punya `authorize()`, `rules()`, dan `messages()`
- [ ] ViewModel punya semua getter yang diperlukan view
- [ ] ServiceProvider bind Interface ke Implementation
- [ ] ServiceProvider load routes dari `routes.php`
- [ ] ServiceProvider register Policy via `Gate::policy()`
- [ ] Blade views menggunakan `<x-layouts.app>`, `<x-card>`, `<x-pagination>`, `<x-badge>`, `<x-empty-state>`
- [ ] Form partial menggunakan `<x-form.input>`, `<x-form.select>`, `<x-form.textarea>`, `<x-form.checkbox>`
- [ ] Tombol form menggunakan `<x-button>` — TIDAK ADA raw `<button class="...">` atau `<a class="...">` untuk aksi
- [ ] Flash message via `->with('success'/'error', '...')` di controller — SweetAlert2 toast otomatis, bukan `<x-alert />` (komponen itu hanya untuk inline alert non-flash yang di-pass manual)
- [ ] Tombol hapus menggunakan `onclick="confirmDelete(...)"` — TIDAK ADA form inline dengan `@method('DELETE')`
- [ ] Semua output Blade menggunakan `{{ }}` (bukan `{!! !!}` kecuali trusted)
- [ ] `bootstrap/providers.php` sudah ditambahkan ServiceProvider modul
- [ ] `php artisan migrate` dijalankan

**Jika ada filter (pertanyaan h):**
- [ ] Controller `index()` menggunakan `$request->only([...])` dan `$this->repository->filter($filters)`
- [ ] Repository `filter()` menggunakan `->withQueryString()` agar filter bertahan di pagination
- [ ] Index view: key filter di form (`name="search"` dsb.) cocok dengan `$request->only([...])`

**Jika ada export (pertanyaan i = Ya):**
- [ ] Package `rap2hpoutre/fast-excel` sudah diinstall
- [ ] Route export ditempatkan **sebelum** `Route::resource` di `routes.php`
- [ ] Controller punya method `export()` yang return `StreamedResponse`

**Jika ada file upload (pertanyaan j):**
- [ ] Migration punya kolom `*_path` yang nullable
- [ ] Form `create` dan `edit` punya `enctype="multipart/form-data"`
- [ ] Action menangani upload, update (hapus file lama), dan delete (hapus file)
- [ ] Storage disk `public` sudah di-symlink: `php artisan storage:link`

**Jika RBAC (pertanyaan k = 2):**
- [ ] Package `spatie/laravel-permission` sudah diinstall dan dimigrasikan
- [ ] `User` model punya trait `HasRoles`
- [ ] Policy menggunakan `hasPermissionTo()` bukan `return true`
- [ ] Permissions di-seed di `DatabaseSeeder`

Tampilkan command untuk user:
```bash
php artisan migrate
php artisan route:list --name={module}  # verifikasi routes terdaftar
```
