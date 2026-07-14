# Design System: UI/UX Guideline

## Prinsip Desain

1. **Clean & Minimal** — White space adalah elemen desain. Jangan padatkan layout.
2. **Konsisten** — Semua elemen menggunakan token yang sama (warna, radius, shadow).
3. **Hierarchy Jelas** — Pengguna tahu di mana mereka berada dan apa yang bisa dilakukan.
4. **Feedback Instan** — Setiap aksi punya visual feedback (hover, loading, success/error).
5. **Aksesibel** — Kontras warna WCAG AA minimum, label form selalu ada.

---

## Layout Utama

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR (fixed, 260px)    │  NAVBAR (sticky, h-16)         │
│ ┌──────────────────────┐   │ ┌──────────────────────────┐   │
│ │ 🏢 AppName           │   │ │ Breadcrumb   🔔  Avatar  │   │
│ │──────────────────────│   │ └──────────────────────────┘   │
│ │                      │   │                                │
│ │ ● Dashboard          │   │  PAGE HEADER                   │
│ │                      │   │ ┌──────────────────────────┐   │
│ │ MANAGEMENT           │   │ │ Page Title   [+ Add]     │   │
│ │ ○ Produk             │   │ └──────────────────────────┘   │
│ │ ○ Pesanan            │   │                                │
│ │ ○ Pelanggan          │   │  CONTENT                       │
│ │                      │   │ ┌──────────────────────────┐   │
│ │ SISTEM               │   │ │                          │   │
│ │ ○ Pengguna           │   │ │   Card / Table / Form    │   │
│ │ ○ Pengaturan         │   │ │                          │   │
│ │                      │   │ └──────────────────────────┘   │
│ │──────────────────────│   │                                │
│ │ 👤 Nama User      ▾  │   │                                │
│ └──────────────────────┘   │                                │
└──────────────────────────────────────────────────────────────┘
```

---

## Design Tokens

### Warna

```
Primary     blue-600   (#2563EB)  → button utama, link aktif, focus ring
            blue-50    (#EFF6FF)  → sidebar active background
            blue-700   (#1D4ED8)  → hover state primary

Success     emerald-600 (#059669) → badge, alert, icon sukses
            emerald-50  (#ECFDF5) → alert background
Danger      red-600    (#DC2626)  → button hapus, badge error
            red-50     (#FEF2F2)  → alert background
Warning     amber-500  (#F59E0B)  → badge warning
            amber-50   (#FFFBEB)  → alert background
Info        sky-500    (#0EA5E9)  → badge info
            sky-50     (#F0F9FF)  → alert background

Surface     white      (#FFFFFF)  → card, sidebar, navbar
            gray-50    (#F9FAFB)  → halaman background
Border      gray-200   (#E5E7EB)  → card border, divider
            gray-300   (#D1D5DB)  → input border

Text        gray-900   (#111827)  → heading utama
            gray-700   (#374151)  → body text
            gray-500   (#6B7280)  → label, placeholder, muted
            gray-400   (#9CA3AF)  → section header sidebar
```

### Typography

Font utama: **Inter** (Google Fonts)

```css
/* Di <head> layout */
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

```
text-xs    font-medium  text-gray-500   → label sidebar section, timestamp
text-sm    font-normal  text-gray-700   → body text, table cell, form label
text-sm    font-medium  text-gray-900   → tabel header, badge
text-base  font-semibold text-gray-900  → card title, section header
text-lg    font-semibold text-gray-900  → page section title
text-xl    font-bold    text-gray-900   → halaman title (h1)
text-2xl   font-bold    text-gray-900   → stat card number
```

### Border Radius

```
rounded-lg   (8px)   → button, input, badge
rounded-xl   (12px)  → card utama
rounded-2xl  (16px)  → modal
rounded-full         → avatar, dot indicator
```

### Shadow

```
shadow-sm                → card default
shadow-md                → dropdown, sticky element
shadow-lg                → modal overlay content
```

### Spacing Konsisten

```
Padding card      : p-6 (24px)
Gap grid kolom    : gap-6 (24px)
Gap antar section : space-y-6
Padding tabel cell: px-4 py-3
Height navbar     : h-16 (64px)
Width sidebar     : w-64 (256px)
```

---

## Komponen Inventory

### Layout
| Komponen | File | Keterangan |
|---|---|---|
| App Layout | `layouts/app.blade.php` | Sidebar + Navbar wrapper |
| Auth Layout | `layouts/auth.blade.php` | Center-card untuk login |
| Sidebar | `layouts/_partials/sidebar.blade.php` | Nav tree |
| Navbar | `layouts/_partials/navbar.blade.php` | Top bar |

### Feedback & Status
| Komponen | File | Props |
|---|---|---|
| Alert | `components/alert.blade.php` | type, message (auto-detect session) |
| Badge | `components/badge.blade.php` | type (success/danger/warning/info/default) |
| Empty State | `components/empty-state.blade.php` | title, description, action |

### Container
| Komponen | File | Props |
|---|---|---|
| Card | `components/card.blade.php` | title, slot:actions |
| Stat Card | `components/stat-card.blade.php` | label, value, change, icon |
| Modal | `components/modal.blade.php` | id, title |

### Navigasi
| Komponen | File | Props |
|---|---|---|
| Breadcrumb | `components/breadcrumb.blade.php` | items (array) |
| Pagination | `components/pagination.blade.php` | paginator |

### Form
| Komponen | File | Props |
|---|---|---|
| Input | `components/form/input.blade.php` | name, label, type, required |
| Select | `components/form/select.blade.php` | name, label, options |
| Textarea | `components/form/textarea.blade.php` | name, label, rows |
| Checkbox | `components/form/checkbox.blade.php` | name, label |
| File Upload | `components/form/file.blade.php` | name, label, accept |

### Aksi
| Komponen | File | Props |
|---|---|---|
| Button | `components/button.blade.php` | variant, type, href |
| Action Buttons | (inline di table) | Edit, Detail, Delete |

---

## Button System

```
Variant     Tailwind Classes
---------   ---------------------------------------------------------------
primary     bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500
secondary   bg-white text-gray-700 border border-gray-300 hover:bg-gray-50
danger      bg-red-600 text-white hover:bg-red-700 focus:ring-red-500
warning     bg-amber-500 text-white hover:bg-amber-600
ghost       text-gray-600 hover:bg-gray-100 hover:text-gray-900

Size        Tailwind Classes
---------   ---------------------------
sm          px-3 py-1.5 text-xs
md          px-4 py-2 text-sm          ← default
lg          px-5 py-2.5 text-base

Semua button: rounded-lg font-medium focus:outline-none focus:ring-2 focus:ring-offset-2
              transition-colors duration-150 inline-flex items-center gap-2
```

---

## Badge / Status System

```
Status      Color Class                         Contoh Penggunaan
--------    --------------------------------    ------------------
success     bg-emerald-50 text-emerald-700      Aktif, Selesai, Lunas
danger      bg-red-50 text-red-700              Nonaktif, Dibatalkan, Gagal
warning     bg-amber-50 text-amber-700          Pending, Menunggu
info        bg-sky-50 text-sky-700              Draft, Proses
default     bg-gray-100 text-gray-600           Tidak diketahui
```

Semua badge: `px-2.5 py-0.5 text-xs font-medium rounded-full`

---

## Form Design Pattern

### Input Group

```
Label (text-sm font-medium text-gray-700, mb-1)
Input (
  w-full rounded-lg border-gray-300 shadow-sm text-sm
  focus:ring-2 focus:ring-blue-500 focus:border-blue-500
  disabled:bg-gray-50 disabled:cursor-not-allowed
  [error]: border-red-500 focus:ring-red-500
)
Error message (text-xs text-red-600 mt-1, dengan ikon ⚠)
Helper text (text-xs text-gray-500 mt-1)
```

### Form Layout

```
Single column  → sm:max-w-md   (form sederhana, login)
Two column     → grid grid-cols-1 sm:grid-cols-2 gap-6
Three column   → grid grid-cols-1 sm:grid-cols-3 gap-4
Full width     → sm:col-span-2 / sm:col-span-3
```

### Form Actions

```
Selalu di bawah form, rata kanan:
<div class="mt-8 pt-6 border-t border-gray-200 flex justify-end gap-3">
  [Batal]    → secondary button
  [Simpan]   → primary button (submit)
</div>
```

---

## Table Design Pattern

```
┌─────────────────────────────────────────────────────────┐
│  Search bar kiri  │  Filter button  │  + Tambah (kanan) │  ← toolbar
├──────┬────────────┬────────┬────────┬───────────────────┤
│  #   │  NAMA      │ STATUS │ HARGA  │  AKSI             │  ← thead (gray-50)
├──────┼────────────┼────────┼────────┼───────────────────┤
│  1   │  Produk A  │ ●Aktif │ Rp 50k │  Detail Edit Hapus│  ← tbody row
│  2   │  Produk B  │ ○Nonaktif│ Rp 30k│ Detail Edit Hapus│  ← hover:bg-gray-50
└──────────────────────────────────────────────────────────┘
Menampilkan 1–15 dari 84 data          [← 1  2  3  4 →]    ← pagination
```

Spesifikasi:
- `thead`: `bg-gray-50 text-xs font-medium text-gray-500 uppercase tracking-wider`
- `tbody row`: `hover:bg-gray-50 transition-colors duration-100`
- `td`: `px-4 py-3 text-sm text-gray-700 whitespace-nowrap`
- `td action`: `text-right` dengan 3 tombol inline: Detail (blue), Edit (amber), Hapus (red)
- Tombol Hapus menggunakan `onclick="confirmDelete(url, label)"` dari SweetAlert2 (bukan form inline atau browser `confirm()`)

---

## Sidebar Navigation Design

```css
/* Item aktif */
.nav-item-active {
  @apply bg-blue-50 text-blue-700 font-medium;
}

/* Item normal */
.nav-item {
  @apply text-gray-600 hover:bg-gray-50 hover:text-gray-900;
}

/* Semua item */
.nav-item-base {
  @apply flex items-center gap-3 px-3 py-2 text-sm rounded-lg
         transition-colors duration-150 cursor-pointer;
}

/* Section header */
.nav-section {
  @apply px-3 pt-4 pb-1 text-xs font-semibold text-gray-400 uppercase tracking-wider;
}
```

---

## Icon System

Gunakan **Heroicons** (inline SVG, 20px / 24px). Install via:
```bash
composer require blade-ui-kit/blade-heroicons
```

Penggunaan:
```blade
<x-heroicon-o-home class="w-5 h-5" />          {{-- outline, 20px --}}
<x-heroicon-s-home class="w-5 h-5" />          {{-- solid --}}
<x-heroicon-m-home class="w-4 h-4" />          {{-- mini, 16px --}}
```

Icon yang wajib digunakan per konteks:
```
Dashboard    → heroicon-o-home
Produk       → heroicon-o-cube
Pesanan      → heroicon-o-shopping-bag
Pelanggan    → heroicon-o-users
Laporan      → heroicon-o-chart-bar
Pengaturan   → heroicon-o-cog-6-tooth
Pengguna     → heroicon-o-user-circle
Tambah       → heroicon-o-plus
Edit         → heroicon-o-pencil-square
Hapus        → heroicon-o-trash
Detail/Lihat → heroicon-o-eye
Cari         → heroicon-o-magnifying-glass
Filter       → heroicon-o-funnel
Export       → heroicon-o-arrow-down-tray
Notifikasi   → heroicon-o-bell
Logout       → heroicon-o-arrow-right-on-rectangle
Chevron kanan→ heroicon-m-chevron-right
```

---

## Dark Mode

Tailwind `darkMode: 'class'` sudah diaktifkan di `tailwind.config.js`. Toggle via Alpine.js:

```blade
<!-- Toggle button di navbar -->
<button @click="$store.theme.toggle()" class="...">
    <x-heroicon-o-moon x-show="$store.theme.isDark" class="w-5 h-5" />
    <x-heroicon-o-sun x-show="!$store.theme.isDark" class="w-5 h-5" />
</button>
```

```js
// resources/js/app.js
Alpine.store('theme', {
    isDark: localStorage.getItem('theme') === 'dark',
    toggle() {
        this.isDark = !this.isDark;
        document.documentElement.classList.toggle('dark', this.isDark);
        localStorage.setItem('theme', this.isDark ? 'dark' : 'light');
    },
    init() {
        document.documentElement.classList.toggle('dark', this.isDark);
    }
});
```

---

## Responsive Breakpoints

```
Mobile   < 640px   → sidebar tersembunyi (toggle), layout 1 kolom
Tablet   640–1024px → sidebar collapsed atau off-canvas
Desktop  > 1024px  → sidebar visible penuh (w-64)
```

Sidebar mobile toggle:
```blade
<!-- Tombol hamburger di navbar (hanya mobile) -->
<button @click="sidebarOpen = !sidebarOpen" class="lg:hidden ...">
    <x-heroicon-o-bars-3 class="w-6 h-6" />
</button>
```

---

## Accessibility Checklist

- [ ] Semua input punya `<label>` atau `aria-label`
- [ ] Form error punya `aria-describedby` ke error message
- [ ] Button icon-only punya `aria-label`
- [ ] Modal punya `role="dialog"` dan `aria-modal="true"`
- [ ] Focus management di modal (trap focus saat buka)
- [ ] Kontras teks minimum 4.5:1 (WCAG AA)
- [ ] Tombol hapus punya konfirmasi (modal, bukan browser confirm)
