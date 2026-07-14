---
name: laravel-blade
description: >-
  Scaffold, extend, atau review proyek Laravel Blade fullstack dengan
  Clean Architecture + Modular Monolith. Lazy loading aman, query efisien,
  dan struktur mudah di-maintain. Invoke dengan: new-project, new-module, atau review.
---

# Laravel Blade — Clean Architecture + Modular Monolith

Skill ini membantu membangun, memperluas, dan mereview proyek **Laravel Blade fullstack** yang menerapkan **Clean Architecture** dengan pendekatan **Modular Monolith**. Hasilkan kode yang production-ready, konsisten, dan langsung bisa dijalankan.

## Cara Invoke

```
/laravel-blade new-project [NamaProyek]
/laravel-blade new-module [NamaModul] [NamaEntity]
/laravel-blade review
```

Jika argumen tidak diberikan, tanyakan yang diperlukan sebelum memulai.

---

## Prinsip Arsitektur

Baca `references/architecture.md` untuk penjelasan lengkap. Ringkasan:

| Layer | Tanggung Jawab | Boleh Bergantung Ke |
|---|---|---|
| **Domain** | Eloquent Model, Events, Policies, Exceptions | Laravel Framework (Eloquent) |
| **Application** | Actions (Use Cases), DTOs, Repository Interfaces | Domain |
| **Infrastructure** | Eloquent Repository implementations, External services | Application, Domain |
| **Presentation** | Controllers, Form Requests, ViewModels | Application, Domain (hanya type-hint/wrap entity yang sudah di-fetch, TIDAK BOLEH query langsung) |

**Modular:** Setiap modul bisnis punya folder sendiri `app/Modules/{ModuleName}/` dengan 4 sub-layer di dalamnya.

**Pengecualian Presentation → Domain:** ViewModel boleh `use` Domain Model untuk type-hint constructor (ia hanya membungkus entity yang sudah diambil oleh Repository). Form Request boleh `use` Domain Model untuk referensi class di Policy check (`$this->user()->can('update', {Entity}::class)`). Yang **dilarang**: Presentation memanggil method query Eloquent (`::find()`, `::findOrFail()`, `::where()`, dst.) langsung — itu WAJIB lewat Repository Interface.

---

## Perintah: `new-project`

### Langkah eksekusi

1. Tanyakan (jika belum ada di args) — **tanyakan semua poin ini sebelum membuat file apapun**:

   **a. Nama proyek** (mis. `TokoOnline`, `SistemInventory`)

   **b. Versi Laravel** — tampilkan pilihan berikut:
   ```
   Pilih versi Laravel:
   [1] Laravel 12  (PHP 8.2+, terbaru) ← direkomendasikan
   [2] Laravel 11  (PHP 8.2+, LTS sebelumnya)
   ```

   **c. Database provider:**
   ```
   Pilih database:
   [1] MySQL / MariaDB  ← default
   [2] PostgreSQL
   [3] SQLite           (cocok untuk development/prototype)
   ```

   **d. CSS Framework:**
   ```
   Pilih CSS Framework:
   [1] Tailwind CSS  ← default (sudah include via Vite + Laravel)
   [2] Bootstrap 5
   ```

   **e. Autentikasi:**
   ```
   Pilih autentikasi:
   [1] Laravel Breeze (Blade stack) ← recommended
   [2] Tidak ada (custom nanti)
   ```

2. Baca `references/packages.md` → bagian **Compatibility Matrix** untuk package yang sesuai.

3. Baca `templates/project-structure.md` untuk struktur folder dan isi file lengkap.

4. Jalankan perintah berikut **secara berurutan**:

   ```bash
   # Buat project Laravel baru
   composer create-project laravel/laravel {ProjectName} "^{version}.0"
   cd {ProjectName}

   # Jika pilih Tailwind
   npm install

   # Jika pilih Breeze
   composer require laravel/breeze --dev
   php artisan breeze:install blade
   npm install && npm run build

   # Jika pilih Bootstrap
   npm install bootstrap @popperjs/core
   ```

5. Baca `templates/sweetalert-config.md` untuk konfigurasi SweetAlert2.

6. Buat **seluruh** file berikut (isi dengan kode nyata, bukan placeholder).
   Untuk setiap file Blade dan JS, baca juga `templates/ui-components.md` dan `references/design-system.md`:
   - `app/Providers/AppServiceProvider.php` — wajib: `preventLazyLoading()` di `boot()`
   - `app/Http/Controllers/Controller.php` — base controller
   - `app/Modules/.gitkeep` — placeholder direktori Modules
   - `resources/js/swal.js` — **SweetAlert2 config** (dari `sweetalert-config.md` §File 1)
   - `resources/css/sweetalert.css` — **SweetAlert2 custom CSS** (dari `sweetalert-config.md` §File 2)
   - `resources/js/app.js` — import Alpine + swal (dari `sweetalert-config.md` §File 3)
   - `resources/css/app.css` — Tailwind + import sweetalert.css (dari `sweetalert-config.md` §File 4)
   - `resources/views/layouts/app.blade.php` — layout utama dengan `<div id="flash-messages">` hidden
   - `resources/views/layouts/auth.blade.php` — layout auth (jika Breeze)
   - `resources/views/layouts/_partials/navbar.blade.php`
   - `resources/views/layouts/_partials/sidebar.blade.php`
   - `resources/views/components/card.blade.php`
   - `resources/views/components/stat-card.blade.php`
   - `resources/views/components/badge.blade.php`
   - `resources/views/components/button.blade.php`
   - `resources/views/components/empty-state.blade.php`
   - `resources/views/components/breadcrumb.blade.php`
   - `resources/views/components/pagination.blade.php`
   - `resources/views/components/modal.blade.php` — untuk kebutuhan non-konfirmasi
   - `resources/views/components/alert.blade.php` — untuk inline alert non-flash (lihat aturan §5 di bawah)
   - `resources/views/components/form/input.blade.php`
   - `resources/views/components/form/select.blade.php`
   - `resources/views/components/form/textarea.blade.php`
   - `resources/views/components/form/checkbox.blade.php`
   - `resources/views/components/form/file.blade.php` — dipakai saat `new-module` menambahkan field file/gambar
   - `resources/views/dashboard/index.blade.php` — halaman dashboard dengan stat cards
   - `app/Http/Controllers/DashboardController.php`
   - `vite.config.js`
   - `.env.example`
   - `.gitignore`

7. Tampilkan ringkasan pilihan:
   ```
   Laravel Version  : 12.x
   PHP Minimum      : 8.2
   Database         : MySQL
   CSS Framework    : Tailwind CSS
   Autentikasi      : Laravel Breeze (Blade)
   ```

8. Tampilkan perintah selanjutnya:
   ```bash
   cp .env.example .env
   php artisan key:generate
   # Edit DB_* di .env, lalu:
   php artisan migrate
   npm run dev   # atau npm run build untuk production
   php artisan serve
   ```

### Aturan wajib saat scaffold

- `AppServiceProvider::boot()` WAJIB berisi `Model::preventLazyLoading(! app()->isProduction())`.
- Layout `app.blade.php` WAJIB include `@stack('scripts')` dan `@stack('styles')`.
- Setiap Blade component WAJIB menggunakan `<x-component-name>` syntax (bukan `@include`).
- Route dashboard WAJIB dilindungi middleware `auth`.
- Jika pilih Tailwind v3: `tailwind.config.js` WAJIB include path `./app/**/*.php` di array `content`. Tailwind v4: tidak perlu — Blade path di-detect otomatis oleh `@tailwindcss/vite`.
- Jika pilih Bootstrap: install via npm, bukan CDN — untuk production build yang proper.

---

## Perintah: `new-module`

### Langkah eksekusi

1. Tanyakan (jika belum ada di args) — **tanyakan semua poin sebelum membuat file apapun**:

   **a. Nama modul** (mis. `Products`, `Orders`, `Users`) — PascalCase, plural

   **b. Nama entity** (mis. `Product`, `Order`, `User`) — PascalCase, singular

   **c. Fields** — tanyakan satu per satu:
   ```
   Daftarkan field entity {EntityName} (ketik 'done' jika selesai):
   Format: nama_field:tipe (mis. name:string, price:decimal, is_active:boolean)
   Tipe yang tersedia: string, text, integer, bigInteger, decimal, boolean, date, datetime, json, foreignId
   ```

   **d. Operasi yang dibutuhkan** (centang semua yang relevan):
   ```
   [ ] Create   [ ] Read (List + Detail)   [ ] Update   [ ] Delete
   ```

   **e. Apakah perlu Soft Delete?** (boolean)

   **f. Apakah perlu audit timestamps?** (`created_by`, `updated_by`) — memerlukan `created_by` dan `updated_by` columns.

   **g. Relasi (opsional):**
   ```
   Apakah entity ini punya relasi? (mis. belongsTo:Category, hasMany:Review)
   Ketik 'none' jika tidak ada.
   ```

   **h. Filter di halaman list (opsional):**
   ```
   Selain search keyword, apakah ada filter lain di halaman index?
   Contoh: status (Aktif/Nonaktif), kategori (dropdown), tanggal dibuat (date range)
   Ketik field-field filter atau 'none'.
   ```
   > Jika ada filter: Controller WAJIB pass nilai filter aktif kembali ke view via `compact()` agar input filter bersifat sticky (nilai tidak hilang setelah submit). Gunakan `request('field')` atau `old('field')` di Blade untuk pre-fill nilai.

   **i. Apakah perlu fitur export data?**
   ```
   [1] Ya — tambahkan tombol Export Excel/CSV di halaman index
   [2] Tidak
   ```

   **j. Apakah ada field file atau gambar?**
   ```
   Contoh: photo:image, document:file, attachment:file
   Ketik 'none' jika tidak ada.
   ```

   **k. Level otorisasi:**
   ```
   [1] Semua user terautentikasi boleh akses (return true)  ← default
   [2] RBAC via spatie/laravel-permission — Policy pakai hasPermissionTo()
   ```

2. Baca `templates/module-slice.md` untuk pola lengkap tiap layer.
   - Jika ada filter (h): terapkan pola dari §Pola Filter Multi-Kolom
   - Jika ada export (i = Ya): terapkan pola dari §Pola Export Excel/CSV
   - Jika ada file upload (j): terapkan pola dari §Pola File Upload
   - Jika RBAC (k = 2): terapkan pola dari §Pola Role & Permission
   - Jika ada relasi (g): terapkan pola dari §Pola Relasi di Form
   - Operasi yang **tidak** dipilih di poin (d): jangan buat Action/Request/route/tombol untuk operasi itu. Contoh — jika Delete tidak dipilih: skip `Delete{Entity}Action.php`, route `destroy`, method `destroy()` di Controller, dan tombol hapus di view. Jika hanya Read yang dipilih (read-only module): skip seluruh `Application/Actions/`, `Presentation/Requests/`, dan halaman create/edit.

3. Buat file-file berikut sesuai pilihan:

#### Struktur folder satu modul

```
app/Modules/{Module}/
├── {Module}ServiceProvider.php
├── routes.php
├── Domain/
│   ├── Models/
│   │   └── {Entity}.php                ← Eloquent model
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

4. Daftarkan `{Module}ServiceProvider` di `bootstrap/providers.php`.

5. Tampilkan migration command:
   ```bash
   php artisan migrate
   ```

### Aturan wajib saat new-module

- Eloquent model WAJIB mendefinisikan `$fillable` atau `$guarded` — TIDAK BOLEH dibiarkan default.
- Eloquent model WAJIB mendefinisikan relationships secara eksplisit dengan return type hints.
- Repository Implementation WAJIB selalu menggunakan `with()` untuk eager load relasi yang diperlukan — TIDAK BOLEH mengandalkan lazy loading.
- Action class WAJIB inject Repository Interface (bukan implementation) — dependency inversion.
- DTO WAJIB menggunakan `readonly` properties (PHP 8.1+) — murni data object, TIDAK BOLEH import class dari Presentation layer.
- Form Request WAJIB punya method `toDTO()` yang menghasilkan DTO — factory dipindah ke Presentation agar DTO tetap bebas dari Http concern.
- Controller WAJIB inject Actions dan Repository via constructor — TIDAK BOLEH instantiate langsung.
- Controller WAJIB mengembalikan `view()` atau `redirect()` — TIDAK BOLEH mengembalikan raw data.
- Form Requests WAJIB mendefinisikan `rules()` dan `messages()` — validasi TIDAK BOLEH ada di controller.
- Jika Form Request `authorize()` butuh entity dari database (mis. untuk Policy check saat update), WAJIB inject Repository Interface via constructor (`parent::__construct()` tetap dipanggil) — TIDAK BOLEH panggil `{Entity}::find()`/`findOrFail()` langsung.
- ViewModel WAJIB menyediakan method getter dengan formatting yang tepat (mis. `getFormattedPrice()`).
- Blade views WAJIB menggunakan komponen `<x-card>` dan `<x-pagination>` yang sudah dibuat. Flash message cukup via `->with()` di controller — SweetAlert2 toast muncul otomatis.
- Routes WAJIB menggunakan `Route::resource()` dan dilindungi middleware `auth` (kecuali public).
- ServiceProvider WAJIB bind Repository Interface ke Implementation via `$this->app->bind()`.
- Setiap Action WAJIB dispatch Event setelah operasi berhasil.

---

## Perintah: `review`

### Langkah eksekusi

1. Baca file-file berikut **secara berurutan berdasarkan prioritas**:
   1. `app/Providers/AppServiceProvider.php` — cek `preventLazyLoading()`
   2. Semua `*Controller.php` di `app/Modules/*/Presentation/Controllers/` — cek business logic yang bocor
   3. Semua `*Action.php` di `app/Modules/*/Application/Actions/` — cek inject Interface vs konkret
   4. Semua `*Repository.php` di `app/Modules/*/Infrastructure/Repositories/` — cek eager loading & N+1
   5. Semua `*.blade.php` di `resources/views/` — cek `{!! !!}` dan raw Model yang di-pass langsung
   6. Semua `*DTO.php` — cek tidak ada import dari Presentation layer
2. Periksa setiap item dalam checklist di bawah.
3. Laporkan temuan:

```
## Review: Laravel Clean Architecture

### Pelanggaran Dependency Rule
- [file]: [masalah] → [rekomendasi]

### Pelanggaran Lazy Loading (N+1 Risk)
- [file]: [masalah] → [rekomendasi]

### Pelanggaran Action Pattern
- [file]: [masalah] → [rekomendasi]

### Masalah Kualitas Kode
- [file:baris]: [masalah] → [rekomendasi]

### Hal yang sudah benar ✓
- ...
```

### Checklist review

**Dependency Rule**
- [ ] Presentation tidak akses Repository langsung (harus lewat Action atau inject Interface)
- [ ] Presentation (Controller, Form Request, ViewModel) tidak memanggil method query Eloquent (`::find()`, `::findOrFail()`, `::where()`, dst.) langsung ke Domain Model — WAJIB lewat Repository Interface. Boleh `use` Domain Model hanya untuk type-hint atau referensi class (mis. Policy check).
- [ ] Action tidak inject Eloquent Model langsung (harus lewat Repository Interface)
- [ ] Domain/Models tidak import class dari Application, Infrastructure, atau Presentation
- [ ] Controller tidak berisi business logic (hanya delegate ke Action)

**Lazy Loading & Performance**
- [ ] `AppServiceProvider::boot()` memanggil `Model::preventLazyLoading(!app()->isProduction())`
- [ ] Repository implementation selalu pakai `with()` untuk eager load (tidak ada $model->relation di luar model)
- [ ] Tidak ada foreach loop yang trigger query N+1 (setiap iterasi memicu query relasi)
- [ ] Halaman list menggunakan `paginate()` — TIDAK BOLEH `get()` yang load semua records
- [ ] Large dataset processing menggunakan `lazy()` atau `chunk()`, bukan `all()`

**Action Pattern**
- [ ] Setiap use case punya satu Action class (Single Responsibility)
- [ ] Action inject Repository Interface (bukan implementation konkret)
- [ ] Action dispatch Event setelah operasi berhasil
- [ ] Controller tidak punya lebih dari 10 baris logic per method

**Repository Pattern**
- [ ] Repository Interface di Application layer (`Application/Contracts/`)
- [ ] Repository Implementation di Infrastructure layer
- [ ] ServiceProvider bind Interface ke Implementation
- [ ] Repository tidak berisi business logic (hanya query + return data)

**DTO & ViewModel**
- [ ] DTO adalah `readonly` (PHP 8.1+)
- [ ] DTO punya static method `fromRequest()` untuk hydrate dari Form Request
- [ ] ViewModel tidak akses database (hanya transform data yang sudah ada)
- [ ] Blade views hanya menerima ViewModel atau array sederhana (bukan Model mentah)

**Form Request & Validasi**
- [ ] Semua input user divalidasi via Form Request (bukan di controller)
- [ ] Form Request mendefinisikan `messages()` dengan pesan Indonesia
- [ ] Tidak ada `$request->all()` yang tidak divalidasi di controller

**Blade Views**
- [ ] Views tidak berisi business logic (hanya display)
- [ ] Semua output di-escape dengan `{{ }}` — TIDAK BOLEH `{!! !!}` kecuali trusted HTML
- [ ] Flash message ditampilkan via SweetAlert2 toast (session `->with()`, bukan `<x-alert>`) — `<x-alert>` hanya untuk inline alert non-flash yang di-pass manual
- [ ] Pagination menggunakan `<x-pagination :paginator="$items" />`

---

## Konvensi Penamaan

Baca `references/conventions.md` untuk detail. Ringkasan:

| Artifact | Pola | Contoh |
|---|---|---|
| Modul | `PascalCase`, plural | `Products`, `Orders` |
| Entity (Eloquent) | `PascalCase`, singular | `Product`, `Order` |
| Action | `{Verb}{Entity}Action` | `CreateProductAction` |
| DTO | `{Verb}{Entity}DTO` | `CreateProductDTO` |
| Repository Interface | `{Entity}RepositoryInterface` | `ProductRepositoryInterface` |
| Repository Impl | `Eloquent{Entity}Repository` | `EloquentProductRepository` |
| Controller | `{Entity}Controller` | `ProductController` |
| Form Request | `{Verb}{Entity}Request` | `CreateProductRequest` |
| ViewModel | `{Entity}ViewModel` | `ProductViewModel` |
| Event | `{Entity}{PastTense}` | `ProductCreated`, `ProductDeleted` |
| Policy | `{Entity}Policy` | `ProductPolicy` |
| Table (migration) | `snake_case`, plural | `products`, `order_items` |
| Route name | `{module}.{action}` | `products.index`, `products.store` |
| View path | `{module}.{entity}.{action}` | `products.product.index` |

---

## Berkas Pendukung

- `references/architecture.md` — penjelasan mendalam Clean Architecture di Laravel, lazy loading strategies
- `references/conventions.md` — naming convention, PSR-12, code style Laravel
- `references/packages.md` — Composer packages yang dipakai dan versi yang direkomendasikan
- `references/design-system.md` — **Design System lengkap**: color tokens, typography, layout, komponen inventory, icon system, responsive breakpoints, aksesibilitas
- `templates/project-structure.md` — template scaffold project lengkap: layout, sidebar, navbar, base components
- `templates/module-slice.md` — template kode lengkap untuk satu module (CRUD penuh) — PHP + Blade
- `templates/ui-components.md` — **Component Library lengkap**: Card, StatCard, Badge, Button, EmptyState, Breadcrumb, Pagination, Form components (Input, Select, Textarea, Checkbox)
- `templates/sweetalert-config.md` — **SweetAlert2 custom theme**: konfigurasi JS, CSS kustom, panduan penggunaan toast + dialog konfirmasi

### Aturan UI/UX Wajib saat Generate Kode

Setiap kali membuat atau memodifikasi Blade views:

1. **Baca `references/design-system.md` SEBELUM menulis Blade** — gunakan token warna, class, dan struktur yang sudah ditetapkan.
2. **Baca `templates/ui-components.md`** — gunakan komponen yang sudah ada (`<x-card>`, `<x-badge>`, `<x-button>`, `<x-form.input>`, dst).
3. **Baca `templates/sweetalert-config.md`** — semua popup/notifikasi WAJIB menggunakan SweetAlert2.
4. Layout WAJIB menggunakan `<x-layouts.app title="..." :breadcrumbs="[...]">`.
5. Flash message (success/error/warning/info) WAJIB via `->with('key', 'pesan')` di controller — SweetAlert2 toast muncul otomatis dari session. TIDAK BOLEH pakai `<x-alert />` untuk flash session atau inline HTML alert manual sebagai penggantinya. (Komponen `<x-alert type="..." message="..." />` sendiri tetap boleh dipakai untuk inline alert **non-flash** yang di-pass manual, mis. peringatan statis di dalam form — lihat `templates/ui-components.md`.)
6. Status/label WAJIB menggunakan `<x-badge type="success|danger|warning|info">`.
7. Tombol hapus WAJIB menggunakan `onclick="confirmDelete(url, label)"` — TIDAK BOLEH `onsubmit="confirm()"` atau `<x-modal>` untuk konfirmasi.
8. Konfirmasi aksi lain (bukan hapus) WAJIB menggunakan `onclick="confirmAction({...}, url, method)"`.
9. Form input WAJIB menggunakan `<x-form.input>`, `<x-form.select>`, dll.
10. Empty state WAJIB menggunakan `<x-empty-state>` bukan paragraf plain.
11. Stat metrics di dashboard WAJIB menggunakan `<x-stat-card>`.
12. Icon WAJIB menggunakan Heroicons (via `blade-ui-kit/blade-heroicons` atau inline SVG dari `design-system.md`).
