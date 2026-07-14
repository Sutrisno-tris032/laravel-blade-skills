# laravel-blade-skills

Claude Code skill plugin untuk pengembangan **Laravel Blade Fullstack** dengan **Clean Architecture** dan **Modular Monolith** approach.

## Skill yang Tersedia

### `laravel-blade`

Scaffold, extend, atau review proyek Laravel Blade fullstack menggunakan Clean Architecture dengan pendekatan modular monolith. Setiap modul bisnis berdiri sendiri, query efisien via eager loading, dan N+1 diproteksi secara otomatis.

**Perintah:**

```
/laravel-blade new-project TokoOnline
/laravel-blade new-module Products Product
/laravel-blade review
```

| Perintah | Fungsi |
|---|---|
| `new-project [NamaProyek]` | Scaffold project Laravel baru dengan struktur Clean Architecture |
| `new-module [Modul] [Entity]` | Tambah satu module lengkap (CRUD) ke project yang ada |
| `review` | Review kode existing terhadap aturan Clean Architecture |

Saat `new-project`, skill akan menanyakan:
- Versi Laravel (12 / 11)
- Database (MySQL / PostgreSQL / SQLite)
- CSS Framework (Tailwind CSS / Bootstrap 5)
- Autentikasi (Laravel Breeze / Tidak ada)

Saat `new-module`, skill akan menanyakan:
- Nama modul dan entity
- Field-field entity (nama, tipe)
- Operasi yang dibutuhkan (Create / Read / Update / Delete)
- Soft Delete (ya/tidak)
- Audit timestamps — `created_by` / `updated_by` (ya/tidak)
- Relasi (belongsTo, hasMany, dll.)
- Filter tambahan di halaman list (status, kategori, date range, dll.)
- Export data ke Excel/CSV (ya/tidak)
- Field file atau gambar (ya/tidak)
- Level otorisasi (semua user auth / RBAC via spatie/laravel-permission)

## Stack Teknologi

- **PHP 8.2+**
- **Laravel 12 / 11** — framework
- **Blade** — template engine bawaan Laravel
- **Eloquent ORM** — dengan Repository Pattern untuk abstraksi query
- **Tailwind CSS** (default) atau **Bootstrap 5** — dipilih saat `new-project`
- **Alpine.js** — lightweight JS (sudah termasuk)
- **Vite** — asset bundler Laravel default
- **Pest** — modern PHP testing framework

## Arsitektur

Setiap modul mengikuti 4-layer Clean Architecture:

```
app/Modules/{ModuleName}/
├── Domain/         ← Eloquent Models, Events, Policies, Exceptions
├── Application/    ← Actions (Use Cases), DTOs, Repository Interfaces
├── Infrastructure/ ← Eloquent Repository Implementations
└── Presentation/   ← Controllers, Form Requests, ViewModels
```

**Dependency Rule:** Presentation → Application → Domain ← Infrastructure

**Lazy Loading Protection:**
- Development: `Model::preventLazyLoading(true)` — throw exception jika N+1 terjadi
- Production: log warning, tidak crash
- Repository selalu gunakan `with()` untuk eager loading eksplisit

## Fitur Utama

- **Action Pattern** — satu class per use case, SRP diterapkan ketat
- **Repository Pattern** — abstraksi Eloquent, mudah di-test dan di-swap
- **DTO (readonly)** — data transfer immutable antar layer
- **ViewModel** — format data untuk tampilan, logic presentasi terpusat
- **Lazy Collection** — `lazy()` untuk dataset besar, hemat memori
- **Event-Driven** — setiap Action dispatch Event untuk decoupling side effects
- **Form Request** — validasi dan otorisasi di luar controller
- **Blade Components** — `<x-card>`, `<x-pagination>`, `<x-badge>`, dan form components yang reusable
- **PSR-12** — code style konsisten dengan `declare(strict_types=1)`

## Instalasi

```
/plugin marketplace add Sutrisno-tris032/laravel-blade-skills
/plugin install laravel-blade@laravel-blade-skills
```

Perintah pertama mendaftarkan repositori GitHub ini sebagai marketplace; perintah kedua mengaktifkan
plugin tersebut beserta skill `laravel-blade` di dalamnya. Untuk memperbarui nanti, jalankan
`/plugin marketplace update-plugin laravel-blade` lalu install ulang.

## Struktur Skill

```
laravel-blade-skills/
├── .claude-plugin/
│   ├── plugin.json                     ← registrasi plugin
│   └── marketplace.json
├── skills/laravel-blade/
│   ├── SKILL.md                        ← entry point, definisi semua perintah
│   ├── references/
│   │   ├── architecture.md             ← Clean Architecture di Laravel, lazy loading strategies
│   │   ├── conventions.md              ← naming convention, PSR-12, code style
│   │   ├── packages.md                 ← Composer packages + compatibility matrix (Tailwind v3/v4)
│   │   └── design-system.md            ← design tokens, komponen inventory, responsif, aksesibilitas
│   └── templates/
│       ├── project-structure.md        ← scaffold project lengkap dengan semua file
│       ├── module-slice.md             ← template CRUD lengkap per module
│       ├── ui-components.md            ← Blade component library lengkap
│       └── sweetalert-config.md        ← SweetAlert2 custom theme & panduan penggunaan
└── README.md
```

---

## Panduan Penggunaan: Dari Business Requirements ke Project Nyata

### Alur Kerja Umum

```
1. Dekomposisi Requirements → Daftar Modul & Entity
          ↓
2. /laravel-blade new-project   → Scaffold project + setup environment
          ↓
3. /laravel-blade new-module    → Tambah modul (ulangi per domain bisnis)
          ↓
4. /laravel-blade review        → Audit kualitas kode (opsional, kapan saja)
```

---

### Langkah 1 — Dekomposisi Requirements

Sebelum invoke skill, ubah narasi bisnis menjadi daftar modul dan entity. Aturan praktis:

- **Satu kata benda bisnis utama = satu entity**
- **Kelompokkan entity yang saling terkait = satu modul**
- **Mulai dari entity tanpa dependency, lanjut ke yang bergantung padanya**

**Contoh requirement:**

> *"Sistem manajemen bengkel. Pelanggan bisa didaftarkan beserta kendaraannya. Mekanik membuat work order per kendaraan, mencatat spare part yang dipakai, lalu menutup order dengan total biaya."*

**Hasil dekomposisi:**

| Urutan | Modul | Entity | Fields Utama | Relasi |
|---|---|---|---|---|
| 1 | `Parts` | `Part` | name, code, price, stock | — |
| 2 | `Customers` | `Customer` | name, phone, email, address | hasMany: Vehicle |
| 3 | `Vehicles` | `Vehicle` | plate_number, brand, model, year, customer_id | belongsTo: Customer |
| 4 | `WorkOrders` | `WorkOrder` | vehicle_id, complaint, status, total_cost | belongsTo: Vehicle |
| 5 | `WorkOrderItems` | `WorkOrderItem` | work_order_id, part_id, qty, unit_price | belongsTo: WorkOrder + Part |

> **Aturan urutan:** Entity tanpa foreign key dibuat lebih dulu. Entity yang punya `belongsTo` ke entity lain dibuat setelahnya.

---

### Langkah 2 — Scaffold Project

```
/laravel-blade new-project BengkelApp
```

Skill akan menanyakan 4 pilihan:

```
Versi Laravel  → [1] Laravel 12
Database       → [1] MySQL
CSS Framework  → [1] Tailwind CSS
Autentikasi    → [1] Laravel Breeze (Blade)
```

Setelah semua file di-generate, jalankan perintah yang diberikan skill:

```bash
cp .env.example .env
php artisan key:generate

# Edit .env — sesuaikan DB_DATABASE, DB_USERNAME, DB_PASSWORD

php artisan migrate
npm run dev        # terminal 1
php artisan serve  # terminal 2
```

Buka `http://localhost:8000` — project sudah bisa diakses dengan halaman login.

---

### Langkah 3 — Tambah Modul Satu per Satu

Invoke `new-module` sesuai urutan dekomposisi (entity tanpa dependency lebih dulu):

#### Modul 1: Parts

```
/laravel-blade new-module Parts Part
```

Jawaban yang diberikan ke skill:

```
Fields    → name:string, code:string, price:decimal, stock:integer
Operasi   → Create, Read, Update, Delete
Soft Del  → Tidak
Audit     → Tidak
Relasi    → none
Filter    → none
Export    → Ya (export stok ke Excel)
File      → Tidak
Otorisasi → [1] Semua user terautentikasi
```

#### Modul 2: Customers

```
/laravel-blade new-module Customers Customer
```

```
Fields    → name:string, phone:string, email:string, address:text
Operasi   → Create, Read, Update, Delete
Soft Del  → Ya
Audit     → Tidak
Relasi    → none  (Vehicle akan punya belongsTo:Customer, bukan sebaliknya)
Filter    → none
Export    → Tidak
File      → Tidak
Otorisasi → [1] Semua user terautentikasi
```

#### Modul 3: Vehicles

```
/laravel-blade new-module Vehicles Vehicle
```

```
Fields    → plate_number:string, brand:string, model:string, year:integer, customer_id:foreignId
Operasi   → Create, Read, Update, Delete
Soft Del  → Tidak
Audit     → Tidak
Relasi    → belongsTo:Customer
Filter    → brand (dropdown)
Export    → Tidak
File      → Ya  (foto kendaraan: photo:image)
Otorisasi → [1] Semua user terautentikasi
```

#### Modul 4: WorkOrders

```
/laravel-blade new-module WorkOrders WorkOrder
```

```
Fields    → vehicle_id:foreignId, complaint:text, status:string, notes:text, total_cost:decimal
Operasi   → Create, Read, Update, Delete
Soft Del  → Ya
Audit     → Ya  (created_by, updated_by)
Relasi    → belongsTo:Vehicle
Filter    → status (Pending/In Progress/Done), tanggal dibuat (date range)
Export    → Ya
File      → Tidak
Otorisasi → [1] Semua user terautentikasi
```

#### Modul 5: WorkOrderItems

```
/laravel-blade new-module WorkOrderItems WorkOrderItem
```

```
Fields    → work_order_id:foreignId, part_id:foreignId, qty:integer, unit_price:decimal
Operasi   → Create, Read, Delete  (Update tidak — item hapus lalu tambah baru)
Soft Del  → Tidak
Audit     → Tidak
Relasi    → belongsTo:WorkOrder, belongsTo:Part
Filter    → none
Export    → Tidak
File      → Tidak
Otorisasi → [1] Semua user terautentikasi
```

Setelah **setiap modul** selesai di-generate, jalankan:

```bash
php artisan migrate
```

---

### Langkah 4 — Review Kode

Setelah semua modul selesai (atau kapan saja selama development):

```
/laravel-blade review
```

Skill membaca semua file PHP dan Blade, lalu melaporkan:
- Pelanggaran dependency rule (mis. Controller akses Repository langsung)
- Risiko N+1 (mis. relasi diakses tanpa eager loading)
- Action yang inject konkret class, bukan Interface
- Blade yang pakai `{!! !!}` tanpa sanitasi

---

### Tips untuk Requirements Kompleks

**Business rule yang tidak sekedar CRUD** — taruh di Domain Model:

```
Requirement: "Work order hanya bisa ditutup jika semua item sudah ada"
→ Buat method WorkOrder::close() di Domain/Models/WorkOrder.php
→ Buat CloseWorkOrderAction di Application/Actions/
→ Tambah route POST /work-orders/{id}/close secara manual setelah new-module selesai
```

**Status workflow (draft → in_progress → completed):**

```
Jawab field: status:string  (skill generate dengan default)
Setelah modul selesai, tambahkan manual:
  - Konstanta STATUS_* di Domain Model
  - Method transition (approve(), reject(), complete()) di Domain Model
  - Action terpisah per transisi
  - Route tambahan di routes.php modul
```

**Modul yang terlalu besar** — pecah menjadi beberapa entity:

```
Jangan: new-module Orders Order  (dengan 20 field)
Lakukan:
  new-module Orders Order         (header: customer, date, status, total)
  new-module OrderLines OrderLine (baris: order_id, product_id, qty, price)
```

**RBAC (multi-role: Admin, Staff, dll.):**

```
Jawab pertanyaan k → [2] RBAC via spatie/laravel-permission
Skill akan:
  - Pakai hasPermissionTo() di Policy
  - Beri instruksi install spatie/laravel-permission
  - Generate seed permissions di DatabaseSeeder
```

**Project existing yang ingin ditambah modul:**

```
/laravel-blade review   ← audit dulu sebelum tambah modul baru
/laravel-blade new-module [Modul] [Entity]
```

---

### Contoh Lain: Sistem Inventory Toko Online

| Urutan | Modul | Entity | Catatan |
|---|---|---|---|
| 1 | `Categories` | `Category` | Tanpa dependency |
| 2 | `Suppliers` | `Supplier` | Tanpa dependency |
| 3 | `Products` | `Product` | belongsTo: Category, Supplier. Ada foto produk. |
| 4 | `Customers` | `Customer` | Tanpa dependency |
| 5 | `Orders` | `Order` | belongsTo: Customer. Status workflow. Export. |
| 6 | `OrderItems` | `OrderItem` | belongsTo: Order + Product |
| 7 | `StockMovements` | `StockMovement` | belongsTo: Product. Audit. Export. |

```
/laravel-blade new-project TokoOnline
/laravel-blade new-module Categories Category
/laravel-blade new-module Suppliers Supplier
/laravel-blade new-module Products Product
/laravel-blade new-module Customers Customer
/laravel-blade new-module Orders Order
/laravel-blade new-module OrderItems OrderItem
/laravel-blade new-module StockMovements StockMovement
/laravel-blade review
```

---

### Contoh Lain: Sistem HR Sederhana

| Urutan | Modul | Entity | Catatan |
|---|---|---|---|
| 1 | `Departments` | `Department` | Tanpa dependency |
| 2 | `Positions` | `Position` | belongsTo: Department |
| 3 | `Employees` | `Employee` | belongsTo: Department + Position. Foto. Soft delete. Audit. |
| 4 | `Attendances` | `Attendance` | belongsTo: Employee. Filter date range. Export. |
| 5 | `LeaveRequests` | `LeaveRequest` | belongsTo: Employee. Status workflow. RBAC. |

```
/laravel-blade new-project HRSystem
/laravel-blade new-module Departments Department
/laravel-blade new-module Positions Position
/laravel-blade new-module Employees Employee
/laravel-blade new-module Attendances Attendance
/laravel-blade new-module LeaveRequests LeaveRequest
/laravel-blade review
```
