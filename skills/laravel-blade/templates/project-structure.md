# Template Scaffold: Project Laravel Baru

Gunakan template ini saat `new-project`. Ganti:
- `{ProjectName}` → nama proyek (mis. `TokoOnline`)
- `{version}` → versi Laravel yang dipilih user (mis. `12`, `11`)
- `{CSSFramework}` → pilihan user: Tailwind atau Bootstrap
- `{AuthMethod}` → Breeze atau Custom

**Jalankan perintah CLI sesuai urutan, BARU buat file custom di bawahnya.**

---

## Langkah 1 — CLI Commands

```bash
# 1. Buat project Laravel
composer create-project laravel/laravel {ProjectName} "^{version}.0"
cd {ProjectName}

# 2a. Jika pilih Tailwind CSS + Breeze:
composer require laravel/breeze --dev
php artisan breeze:install blade
npm install

# 2b. Jika pilih Tailwind CSS tanpa Breeze:
npm install

# 2c. Jika pilih Bootstrap 5:
npm install bootstrap @popperjs/core sass

# 3. Build assets (untuk verifikasi)
npm run build
```

---

## Langkah 2 — Struktur Folder yang Dibuat

```
{ProjectName}/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Controller.php          ← base controller (ada dari Laravel default)
│   │   │   └── DashboardController.php ← BUAT INI
│   │   └── Kernel.php
│   ├── Models/
│   │   └── User.php                    ← sudah ada, MODIFIKASI
│   ├── Modules/                        ← BUAT INI (folder modul)
│   │   └── .gitkeep
│   └── Providers/
│       └── AppServiceProvider.php      ← MODIFIKASI (tambah preventLazyLoading)
│
├── bootstrap/
│   └── providers.php                   ← akan dimodifikasi saat new-module
│
├── database/
│   ├── migrations/
│   │   └── (sudah ada dari default Laravel)
│   └── seeders/
│       └── DatabaseSeeder.php
│
├── resources/
│   ├── css/
│   │   └── app.css                     ← MODIFIKASI sesuai CSS framework
│   ├── js/
│   │   ├── app.js                      ← MODIFIKASI
│   │   └── bootstrap.js
│   └── views/
│       ├── layouts/
│       │   ├── app.blade.php           ← BUAT INI
│       │   ├── auth.blade.php          ← BUAT INI (jika Breeze)
│       │   └── _partials/
│       │       ├── navbar.blade.php    ← BUAT INI
│       │       └── sidebar.blade.php   ← BUAT INI
│       ├── components/
│       │   ├── card.blade.php          ← BUAT INI
│       │   └── pagination.blade.php    ← BUAT INI
│       └── dashboard/
│           └── index.blade.php         ← BUAT INI
│
├── routes/
│   └── web.php                         ← MODIFIKASI
│
├── .env.example                        ← MODIFIKASI sesuai DB pilihan
├── tailwind.config.js                  ← BUAT INI (jika Tailwind)
└── vite.config.js                      ← MODIFIKASI
```

---

## File Contents

### `app/Providers/AppServiceProvider.php`

```php
<?php

declare(strict_types=1);

namespace App\Providers;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Blade;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void {}

    public function boot(): void
    {
        // Daftarkan resources/views/layouts/ sebagai namespace anonymous component "layouts"
        // agar <x-layouts.app> resolve ke resources/views/layouts/app.blade.php
        Blade::anonymousComponentPath(resource_path('views/layouts'), 'layouts');

        // Mencegah N+1 — throw exception di development, log di production
        Model::preventLazyLoading(! app()->isProduction());

        if (app()->isProduction()) {
            Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
                Log::warning('Lazy loading violation detected', [
                    'model'    => get_class($model),
                    'relation' => $relation,
                ]);
            });
        }

        // Mencegah mass assignment tanpa fillable yang jelas (opsional, tapi direkomendasikan)
        Model::preventSilentlyDiscardingAttributes(! app()->isProduction());
    }
}
```

### `app/Http/Controllers/DashboardController.php`

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use Illuminate\View\View;

final class DashboardController extends Controller
{
    public function index(): View
    {
        return view('dashboard.index');
    }
}
```

### `routes/web.php`

```php
<?php

use App\Http\Controllers\DashboardController;
use Illuminate\Support\Facades\Route;

Route::middleware(['web'])->group(function () {
    Route::get('/', fn () => redirect()->route('dashboard'));
});

Route::middleware(['web', 'auth', 'verified'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index'])->name('dashboard');
});

// Module routes akan di-load via ServiceProvider masing-masing modul
// Contoh: App\Modules\Products\ProductsServiceProvider::boot() memanggil $this->loadRoutesFrom(...)
```

### `resources/views/layouts/app.blade.php` (Tailwind CSS)

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}" class="h-full">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ $title ?? config('app.name') }}</title>

    @stack('styles')
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="h-full bg-gray-50 font-sans antialiased">

    <div class="min-h-full">
        {{-- Navbar --}}
        @include('layouts._partials.navbar')

        {{-- Sidebar + Content --}}
        <div class="flex">
            @include('layouts._partials.sidebar')

            <main class="flex-1 p-6">
                {{-- Page Header --}}
                @isset($header)
                    <div class="mb-6">
                        {{ $header }}
                    </div>
                @endisset

                {{-- Flash Messages — dibaca oleh swal.js untuk auto-fire toast --}}
                <div id="flash-messages"
                     data-success="{{ session('success') }}"
                     data-error="{{ session('error') }}"
                     data-warning="{{ session('warning') }}"
                     data-info="{{ session('info') }}"
                     hidden>
                </div>

                {{-- Page Content --}}
                {{ $slot }}
            </main>
        </div>
    </div>

    @stack('scripts')
</body>
</html>
```

### `resources/views/layouts/_partials/navbar.blade.php`

```blade
<nav class="bg-white border-b border-gray-200">
    <div class="max-w-full mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
            <div class="flex items-center">
                <a href="{{ route('dashboard') }}" class="text-xl font-bold text-gray-900">
                    {{ config('app.name') }}
                </a>
            </div>

            <div class="flex items-center gap-4">
                @auth
                    <span class="text-sm text-gray-600">{{ auth()->user()->name }}</span>
                    <form method="POST" action="{{ route('logout') }}">
                        @csrf
                        <button type="submit" class="text-sm text-red-600 hover:text-red-800">
                            Keluar
                        </button>
                    </form>
                @endauth
            </div>
        </div>
    </div>
</nav>
```

### `resources/views/layouts/_partials/sidebar.blade.php`

```blade
<aside class="w-64 min-h-screen bg-white border-r border-gray-200">
    <nav class="p-4 space-y-1">
        <a href="{{ route('dashboard') }}"
           class="flex items-center px-3 py-2 text-sm rounded-md {{ request()->routeIs('dashboard') ? 'bg-blue-50 text-blue-700 font-medium' : 'text-gray-700 hover:bg-gray-100' }}">
            Dashboard
        </a>

        {{-- Tambah item menu per modul di sini --}}
        {{-- Contoh setelah new-module Products: --}}
        {{-- <a href="{{ route('products.index') }}" ...>Produk</a> --}}
    </nav>
</aside>
```

### `resources/views/components/alert.blade.php`

```blade
@props(['type' => null, 'message' => null])

@php
    $types = [
        'success' => ['bg' => 'bg-green-50', 'border' => 'border-green-400', 'text' => 'text-green-800', 'icon' => '✓'],
        'error'   => ['bg' => 'bg-red-50',   'border' => 'border-red-400',   'text' => 'text-red-800',   'icon' => '✕'],
        'warning' => ['bg' => 'bg-yellow-50', 'border' => 'border-yellow-400', 'text' => 'text-yellow-800', 'icon' => '!'],
        'info'    => ['bg' => 'bg-blue-50',  'border' => 'border-blue-400',  'text' => 'text-blue-800',  'icon' => 'i'],
    ];

    $resolvedType    = $type ?? (session('success') ? 'success' : (session('error') ? 'error' : null));
    $resolvedMessage = $message ?? session($resolvedType ?? '');
@endphp

@if ($resolvedType && $resolvedMessage)
    <div class="{{ $types[$resolvedType]['bg'] }} {{ $types[$resolvedType]['border'] }} {{ $types[$resolvedType]['text'] }} border-l-4 p-4 mb-4 rounded"
         role="alert"
         x-data="{ show: true }"
         x-show="show">
        <div class="flex justify-between">
            <p class="text-sm font-medium">
                <span class="mr-1">{{ $types[$resolvedType]['icon'] }}</span>
                {{ $resolvedMessage }}
            </p>
            <button @click="show = false" class="ml-4 text-current opacity-60 hover:opacity-100 text-lg leading-none">&times;</button>
        </div>
    </div>
@endif
```

### `resources/views/components/card.blade.php`

```blade
@props(['title' => null, 'class' => ''])

<div {{ $attributes->merge(['class' => "bg-white rounded-lg shadow-sm border border-gray-200 {$class}"]) }}>
    @if ($title || isset($actions))
        <div class="px-6 py-4 border-b border-gray-200 flex items-center justify-between">
            @if ($title)
                <h3 class="text-lg font-medium text-gray-900">{{ $title }}</h3>
            @endif
            @isset($actions)
                <div class="flex items-center gap-2">
                    {{ $actions }}
                </div>
            @endisset
        </div>
    @endif

    <div class="p-6">
        {{ $slot }}
    </div>
</div>
```

### `resources/views/components/pagination.blade.php`

```blade
@props(['paginator'])

@if ($paginator->hasPages())
    <div class="mt-4 flex items-center justify-between text-sm text-gray-600">
        <p>
            Menampilkan {{ $paginator->firstItem() }}–{{ $paginator->lastItem() }}
            dari {{ $paginator->total() }} data
        </p>
        <div>
            {{ $paginator->links() }}
        </div>
    </div>
@endif
```

### `resources/views/dashboard/index.blade.php`

```blade
<x-layouts.app title="Dashboard">

    <x-slot name="header">
        <h1 class="text-2xl font-semibold text-gray-900">Dashboard</h1>
    </x-slot>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <x-card title="Selamat Datang">
            <p class="text-gray-600">
                Halo, <strong>{{ auth()->user()->name }}</strong>!
                Selamat datang di {{ config('app.name') }}.
            </p>
        </x-card>
    </div>

</x-layouts.app>
```

### `.env.example` (MySQL)

```env
APP_NAME="{ProjectName}"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE={project_name}
DB_USERNAME=root
DB_PASSWORD=

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

### `.env.example` (PostgreSQL)

```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE={project_name}
DB_USERNAME=postgres
DB_PASSWORD=
```

### `.env.example` (SQLite)

```env
DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

### `tailwind.config.js`

```js
/** @type {import('tailwindcss').Config} */
export default {
    content: [
        './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
        './storage/framework/views/*.php',
        './resources/views/**/*.blade.php',
        './app/**/*.php',
    ],
    darkMode: 'class',
    theme: {
        extend: {
            fontFamily: {
                sans: ['Figtree', 'ui-sans-serif', 'system-ui', 'sans-serif'],
            },
        },
    },
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
};
```

### `resources/js/app.js`

```js
import './bootstrap';
import Alpine from 'alpinejs';
import './swal';

window.Alpine = Alpine;

Alpine.store('theme', {
    isDark: localStorage.getItem('theme') === 'dark',
    toggle() {
        this.isDark = !this.isDark;
        document.documentElement.classList.toggle('dark', this.isDark);
        localStorage.setItem('theme', this.isDark ? 'dark' : 'light');
    },
    init() {
        document.documentElement.classList.toggle('dark', this.isDark);
    },
});

Alpine.start();
```

### `resources/css/app.css` (Tailwind)

**Tailwind v3 (Laravel 11):**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* SweetAlert2 custom theme */
@import './sweetalert.css';
```

**Tailwind v4 (Laravel 12):**
```css
@import "tailwindcss";

/* SweetAlert2 custom theme */
@import './sweetalert.css';
```

### `.gitignore`

```
/node_modules
/public/build
/public/hot
/public/storage
/storage/*.key
/vendor
.env
.env.*
!.env.example
*.cache
.phpunit.result.cache
Homestead.json
Homestead.yaml
npm-debug.log
yarn-error.log
/.fleet
/.idea
/.vscode
/database/database.sqlite
```

---

## Perintah Selanjutnya (Tampilkan ke User)

```bash
# 1. Konfigurasi environment
cp .env.example .env
php artisan key:generate

# 2. Edit .env — sesuaikan DB_DATABASE, DB_USERNAME, DB_PASSWORD

# 3. Jalankan migrasi
php artisan migrate

# 4. Start development server (terminal terpisah)
npm run dev

# 5. Start Laravel
php artisan serve

# Buka browser: http://localhost:8000
```

Untuk menambah modul:
```bash
/laravel-blade new-module Products Product
```
