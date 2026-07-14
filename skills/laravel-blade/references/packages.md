# Packages: Composer & NPM

## Compatibility Matrix

| Package | Laravel 11 | Laravel 12 | Kegunaan |
|---|---|---|---|
| `laravel/framework` | `^11.0` | `^12.0` | Core framework |
| `laravel/breeze` | `^2.0` | `^2.3` | Auth scaffolding (dev) |
| `laravel/tinker` | `^2.9` | `^2.10` | REPL |
| `spatie/laravel-permission` | `^6.0` | `^6.10` | Roles & permissions (opsional) |
| `spatie/laravel-activitylog` | `^4.7` | `^4.9` | Audit log (opsional) |
| `blade-ui-kit/blade-heroicons` | `^2.3` | `^2.3` | Heroicons SVG sebagai Blade component |
| `rap2hpoutre/fast-excel` | `^5.0` | `^5.0` | Export Excel/CSV ringan (opsional, jika butuh export) |
| `spatie/laravel-medialibrary` | `^11.0` | `^11.0` | File & image upload dengan konversi otomatis (opsional) |
| `barryvdh/laravel-debugbar` | `^3.13` | `^3.15` | Debug toolbar (dev only) |
| `pestphp/pest` | `^3.0` | `^3.5` | Testing framework |
| `pestphp/pest-plugin-laravel` | `^3.0` | `^3.1` | Pest Laravel integration |
| `mockery/mockery` | `^1.6` | `^1.6` | Mocking di test |
| `fakerphp/faker` | `^1.23` | `^1.23` | Fake data di test/seeder |

## Package NPM (Tailwind CSS)

> **Perhatian Versi:** Laravel 11 default ke Tailwind CSS **v3**, Laravel 12 fresh install default ke Tailwind CSS **v4**. Keduanya didukung tapi sintaksnya berbeda — pilih sesuai versi Laravel yang dipilih.

### Tailwind CSS v3 (Laravel 11)

| Package | Versi | Kegunaan |
|---|---|---|
| `tailwindcss` | `^3.4` | CSS framework |
| `@tailwindcss/forms` | `^0.5` | Form styling plugin |
| `@tailwindcss/typography` | `^0.5` | Prose styling plugin |
| `autoprefixer` | `^10.4` | PostCSS autoprefixer |
| `postcss` | `^8.4` | CSS processor |
| `vite` | `^5.0` | Asset bundler |
| `laravel-vite-plugin` | `^1.0` | Vite + Laravel integration |
| `alpinejs` | `^3.14` | Lightweight JS framework |
| `sweetalert2` | `^11.14` | **Popup & notifikasi** (confirm dialog, toast) |

`app.css` Tailwind v3:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
@import './sweetalert.css';
```

### Tailwind CSS v4 (Laravel 12)

| Package | Versi | Kegunaan |
|---|---|---|
| `tailwindcss` | `^4.0` | CSS framework |
| `@tailwindcss/vite` | `^4.0` | Vite plugin (menggantikan PostCSS di v4) |
| `vite` | `^6.0` | Asset bundler |
| `laravel-vite-plugin` | `^1.0` | Vite + Laravel integration |
| `alpinejs` | `^3.14` | Lightweight JS framework |
| `sweetalert2` | `^11.14` | **Popup & notifikasi** (confirm dialog, toast) |

`app.css` Tailwind v4 — sintaks berubah, **tidak ada `@tailwind` directive**:
```css
@import "tailwindcss";
@import './sweetalert.css';
```

`vite.config.js` Tailwind v4 — plugin masuk ke Vite, bukan PostCSS:
```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
        tailwindcss(),
    ],
});
```

> `tailwind.config.js` **tidak diperlukan** di v4 — konfigurasi pindah ke `app.css` menggunakan `@theme` directive jika perlu kustomisasi.

## Package NPM (Bootstrap 5)

| Package | Versi | Kegunaan |
|---|---|---|
| `bootstrap` | `^5.3` | CSS framework |
| `@popperjs/core` | `^2.11` | Tooltip/dropdown (Bootstrap dep) |
| `sass` | `^1.72` | SCSS processor |
| `vite` | `^5.0` | Asset bundler |
| `laravel-vite-plugin` | `^1.0` | Vite + Laravel integration |
| `alpinejs` | `^3.14` | Lightweight JS framework |
| `sweetalert2` | `^11.14` | **Popup & notifikasi** (confirm dialog, toast) |

---

## Penjelasan Pilihan Package

### Core

**`laravel/framework`** — Tidak perlu diinstall manual, sudah ada saat `composer create-project`.

**`laravel/breeze`** — Scaffolding autentikasi ringan (login, register, forgot password, email verification). Pilih stack `blade` saat install. Install via:
```bash
composer require laravel/breeze --dev
php artisan breeze:install blade
```

### Optional — Roles & Permissions

**`spatie/laravel-permission`** — Package paling populer untuk RBAC di Laravel. Install jika aplikasi butuh multi-role (Admin, Manager, Staff, dll.).

```bash
composer require spatie/laravel-permission
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
php artisan migrate
```

Tambahkan trait ke User model:
```php
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;
}
```

Integrasi di Form Request:
```php
public function authorize(): bool
{
    return $this->user()->hasPermissionTo('create products');
}
```

### Optional — Activity Log

**`spatie/laravel-activitylog`** — Audit trail otomatis. Gunakan jika perlu history perubahan data.

```bash
composer require spatie/laravel-activitylog
php artisan vendor:publish --provider="Spatie\Activitylog\ActivitylogServiceProvider" --tag="activitylog-migrations"
php artisan migrate
```

Di Eloquent Model:
```php
use Spatie\Activitylog\Traits\LogsActivity;
use Spatie\Activitylog\LogOptions;

class Product extends Model
{
    use LogsActivity;

    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->logOnly(['name', 'price', 'stock', 'is_active'])
            ->logOnlyDirty()
            ->dontSubmitEmptyLogs();
    }
}
```

### Export Data (Opsional)

**`rap2hpoutre/fast-excel`** — Export ke Excel/CSV tanpa overhead PHPSpreadsheet. Pilihan terbaik untuk export sederhana (kolom langsung dari model/collection).

```bash
composer require rap2hpoutre/fast-excel
```

Contoh export di Controller:
```php
use Rap2hpoutre\FastExcel\FastExcel;

public function export(): StreamedResponse
{
    return (new FastExcel(
        $this->repository->lazyAll()  // LazyCollection untuk hemat memori
    ))->download('{entities}-' . now()->format('Y-m-d') . '.xlsx');
}
```

Jika butuh kontrol kolom:
```php
return (new FastExcel($this->repository->lazyAll()))
    ->download('{entities}.xlsx', function ({Entity} ${entity}) {
        return [
            'ID'     => ${entity}->id,
            'Nama'   => ${entity}->name,
            'Harga'  => ${entity}->price,
            'Status' => ${entity}->is_active ? 'Aktif' : 'Nonaktif',
        ];
    });
```

### File Upload (Opsional)

**`spatie/laravel-medialibrary`** — Manajemen file/image lengkap dengan konversi ukuran otomatis. Gunakan jika butuh multiple file, konversi thumbnail, atau storage ke S3/cloud.

```bash
composer require spatie/laravel-medialibrary
php artisan vendor:publish --provider="Spatie\MediaLibrary\MediaLibraryServiceProvider" --tag="medialibrary-migrations"
php artisan migrate
```

Untuk upload sederhana (satu file tanpa medialibrary), cukup gunakan `Storage` bawaan Laravel — lihat §Pola File Upload di `templates/module-slice.md`.

---

### Development Tools

**`barryvdh/laravel-debugbar`** — Debug bar di browser untuk inspect queries, time, memory.

```bash
composer require barryvdh/laravel-debugbar --dev
```

Auto-enabled di development via `APP_DEBUG=true`.

### Testing

**`pestphp/pest`** — Modern PHP testing dengan sintaks yang lebih ringkas dari PHPUnit. Diinstall via:
```bash
composer require pestphp/pest pestphp/pest-plugin-laravel --dev
./vendor/bin/pest --init
```

---

## `composer.json` — Template Dasar

```json
{
    "name": "{vendor}/{project-name}",
    "type": "project",
    "description": "",
    "require": {
        "php": "^8.2",
        "laravel/framework": "^11.0",
        "laravel/tinker": "^2.9"
    },
    "require-dev": {
        "fakerphp/faker": "^1.23",
        "laravel/breeze": "^2.0",
        "laravel/pint": "^1.13",
        "mockery/mockery": "^1.6",
        "nunomaduro/collision": "^8.1",
        "pestphp/pest": "^3.0",
        "pestphp/pest-plugin-laravel": "^3.0",
        "barryvdh/laravel-debugbar": "^3.13"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    },
    "scripts": {
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover --ansi"
        ],
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ],
        "post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "@php artisan key:generate --ansi",
            "@php -r \"file_exists('database/database.sqlite') || touch('database/database.sqlite');\"",
            "@php artisan migrate --graceful --ansi"
        ],
        "test": "pest",
        "lint": "pint"
    },
    "extra": {
        "laravel": {
            "dont-discover": []
        }
    },
    "config": {
        "optimize-autoloader": true,
        "preferred-install": "dist",
        "sort-packages": true,
        "allow-plugins": {
            "pestphp/pest-plugin": true,
            "php-http/discovery": true
        }
    },
    "minimum-stability": "stable",
    "prefer-stable": true
}
```

---

## `vite.config.js` — Tailwind CSS

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

## `vite.config.js` — Bootstrap 5

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/scss/app.scss', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

## `resources/css/app.css` — Tailwind

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## `tailwind.config.js`

```js
/** @type {import('tailwindcss').Config} */
export default {
    content: [
        './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
        './storage/framework/views/*.php',
        './resources/views/**/*.blade.php',
        './app/**/*.php',        // untuk kelas Tailwind yang mungkin ada di PHP
    ],
    theme: {
        extend: {},
    },
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
};
```

## `resources/scss/app.scss` — Bootstrap

```scss
// Bootstrap 5
@import 'bootstrap/scss/bootstrap';

// Custom variables (taruh sebelum import)
// $primary: #3490dc;
```

## `resources/js/app.js`

```js
import './bootstrap';
import Alpine from 'alpinejs';

window.Alpine = Alpine;
Alpine.start();
```
