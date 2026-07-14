# SweetAlert2 — Konfigurasi Custom & Panduan Penggunaan

Standar popup dan notifikasi untuk seluruh aplikasi. Wajib dibaca dan diimplementasikan saat `new-project`.

---

## Install

```bash
npm install sweetalert2
```

---

## File 1: `resources/js/swal.js`

File konfigurasi utama — satu tempat untuk semua preset SweetAlert2.

```js
import Swal from 'sweetalert2';

// ─── Base Mixin ─────────────────────────────────────────────────────────────
// Semua preset mewarisi dari base ini (font, class naming, animasi)
const Base = Swal.mixin({
    customClass: {
        popup:          'app-swal-popup',
        title:          'app-swal-title',
        htmlContainer:  'app-swal-html',
        confirmButton:  'app-swal-confirm',
        cancelButton:   'app-swal-cancel',
        denyButton:     'app-swal-deny',
        actions:        'app-swal-actions',
        closeButton:    'app-swal-close',
    },
    buttonsStyling: false,      // matikan styling bawaan, pakai class kita
    showCloseButton: false,
    allowOutsideClick: true,
    showClass: { popup: 'app-swal-enter' },
    hideClass: { popup: 'app-swal-leave' },
});

// ─── Toast ───────────────────────────────────────────────────────────────────
// Notifikasi singkat top-right, dark background, slide dari kanan
export const Toast = Swal.mixin({
    toast: true,
    position: 'top-end',
    showConfirmButton: false,
    timer: 3500,
    timerProgressBar: true,
    customClass: {
        popup:            'app-toast',
        timerProgressBar: 'app-toast-bar',
        title:            'app-toast-title',
    },
    showClass: { popup: 'app-toast-enter' },
    hideClass: { popup: 'app-toast-leave' },
    didOpen: (toast) => {
        toast.addEventListener('mouseenter', Swal.stopTimer);
        toast.addEventListener('mouseleave', Swal.resumeTimer);
    },
});

// ─── Confirm ─────────────────────────────────────────────────────────────────
// Dialog konfirmasi umum — tombol confirm biru
export const Confirm = Base.mixin({
    showCancelButton:  true,
    cancelButtonText:  'Batal',
    confirmButtonText: 'Ya, lanjutkan',
    reverseButtons:    true,
    focusCancel:       true,
});

// ─── ConfirmDelete ────────────────────────────────────────────────────────────
// Dialog hapus data — tombol confirm merah
export const ConfirmDelete = Base.mixin({
    showCancelButton:  true,
    cancelButtonText:  'Batal',
    confirmButtonText: 'Ya, hapus',
    reverseButtons:    true,
    focusCancel:       true,
    customClass: {
        popup:          'app-swal-popup',
        title:          'app-swal-title',
        htmlContainer:  'app-swal-html',
        confirmButton:  'app-swal-danger',   // ← merah
        cancelButton:   'app-swal-cancel',
        actions:        'app-swal-actions',
    },
});

// ─── Expose ke window (untuk penggunaan dari inline Blade onclick) ────────────
window.Swal          = Base;
window.Toast         = Toast;
window.Confirm       = Confirm;
window.ConfirmDelete = ConfirmDelete;

// ─── Global Helpers ──────────────────────────────────────────────────────────

/**
 * Submit form via JS (digunakan setelah konfirmasi).
 * @param {string} url    - action URL
 * @param {string} method - HTTP method (POST/PUT/PATCH/DELETE)
 */
window.submitForm = function (url, method = 'DELETE') {
    const form    = document.createElement('form');
    form.method   = 'POST';
    form.action   = url;
    const token   = document.querySelector('meta[name="csrf-token"]').content;
    form.innerHTML = `
        <input type="hidden" name="_token"  value="${token}">
        <input type="hidden" name="_method" value="${method}">
    `;
    document.body.appendChild(form);
    form.submit();
};

/**
 * Konfirmasi hapus data — tampilkan dialog merah, lalu submit DELETE.
 *
 * Penggunaan di Blade:
 *   <button onclick="confirmDelete('{{ route('products.destroy', $product) }}', '{{ $product->name }}')">
 *       Hapus
 *   </button>
 *
 * @param {string} url   - route destroy
 * @param {string} label - nama/label data yang dihapus
 */
window.confirmDelete = function (url, label = 'data ini') {
    // Escape HTML entities agar label aman di dalam innerHTML
    const safeLabel = label.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');

    ConfirmDelete.fire({
        title: 'Hapus data?',
        html: `
            <div class="flex items-start gap-3 text-left">
                <div class="w-10 h-10 rounded-xl bg-red-50 flex items-center justify-center flex-shrink-0 mt-0.5">
                    <svg class="w-5 h-5 text-red-500" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round"
                              d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"/>
                    </svg>
                </div>
                <div>
                    <p class="text-sm font-semibold text-gray-900">
                        Hapus <span class="text-red-600">${safeLabel}</span>?
                    </p>
                    <p class="text-sm text-gray-500 mt-0.5">
                        Tindakan ini tidak dapat dibatalkan.
                    </p>
                </div>
            </div>
        `,
    }).then(result => {
        if (result.isConfirmed) {
            submitForm(url, 'DELETE');
        }
    });
};

/**
 * Konfirmasi aksi umum (bukan hapus).
 *
 * Penggunaan:
 *   confirmAction({
 *       title: 'Aktifkan produk ini?',
 *       description: 'Produk akan langsung terlihat oleh pelanggan.',
 *       confirmText: 'Ya, aktifkan',
 *   }, '{{ route("products.activate", $product) }}', 'POST');
 */
window.confirmAction = function ({ title, description = '', confirmText = 'Ya, lanjutkan', icon = 'blue' }, url, method = 'POST') {
    const iconColors = {
        blue:   ['bg-blue-50',   'text-blue-500'],
        yellow: ['bg-amber-50',  'text-amber-500'],
        green:  ['bg-emerald-50','text-emerald-500'],
    };
    const [bgColor, textColor] = iconColors[icon] ?? iconColors['blue'];

    Confirm.fire({
        title,
        html: `
            <div class="flex items-start gap-3 text-left">
                <div class="w-10 h-10 rounded-xl ${bgColor} flex items-center justify-center flex-shrink-0 mt-0.5">
                    <svg class="w-5 h-5 ${textColor}" fill="none" viewBox="0 0 24 24" stroke-width="2" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round"
                              d="M12 9v3.75m9-.75a9 9 0 11-18 0 9 9 0 0118 0zm-9 3.75h.008v.008H12v-.008z"/>
                    </svg>
                </div>
                <p class="text-sm text-gray-500 mt-2">${description}</p>
            </div>
        `,
        confirmButtonText: confirmText,
    }).then(result => {
        if (result.isConfirmed) {
            submitForm(url, method);
        }
    });
};

// ─── Auto-fire Flash Session dari layout ────────────────────────────────────
// Layout menyuntikkan pesan flash sebagai data-attribute.
// Script ini membaca dan menampilkan toast saat DOM ready.
document.addEventListener('DOMContentLoaded', () => {
    const el = document.getElementById('flash-messages');
    if (!el) return;

    const iconMap = { success: 'success', error: 'error', warning: 'warning', info: 'info' };

    // Prioritas: success → error → warning → info
    for (const [type, icon] of Object.entries(iconMap)) {
        const msg = el.dataset[type];
        if (msg) {
            Toast.fire({ icon, title: msg });
            break;
        }
    }
});

export { Base as Swal };
```

---

## File 2: `resources/css/sweetalert.css`

Custom CSS yang membuat tampilan SweetAlert2 berbeda dari default.

```css
/* ═══════════════════════════════════════════════════════════════════════════
   SweetAlert2 — Custom Theme
   Design: Inter font · rounded-2xl · dark toast · left-aligned dialog
   ═══════════════════════════════════════════════════════════════════════════ */

/* ─── DIALOG POPUP ─────────────────────────────────────────────────────────── */

.app-swal-popup {
    font-family: 'Inter', ui-sans-serif, system-ui, sans-serif !important;
    border-radius: 1rem !important;
    padding: 1.75rem !important;
    border: 1px solid #E5E7EB !important;
    box-shadow:
        0 20px 48px -8px rgba(0, 0, 0, 0.16),
        0  0    0  1px rgba(0, 0, 0, 0.04) !important;
    max-width: 26rem !important;
    width: 100% !important;
}

/* Backdrop lebih subtle */
.swal2-backdrop-show {
    background: rgba(17, 24, 39, 0.45) !important;
    backdrop-filter: blur(3px) !important;
    -webkit-backdrop-filter: blur(3px) !important;
}

/* Sembunyikan icon default SweetAlert2 — kita pakai custom HTML icon */
.app-swal-popup .swal2-icon {
    display: none !important;
}

.app-swal-title {
    font-size: 1rem !important;
    font-weight: 600 !important;
    color: #111827 !important;
    text-align: left !important;
    padding: 0 0 0.125rem 0 !important;
    margin: 0 !important;
    display: none !important;  /* judul ditaruh di dalam HTML konten */
}

.app-swal-html {
    font-size: 0.875rem !important;
    color: #6B7280 !important;
    text-align: left !important;
    margin: 0 !important;
    padding: 0 !important;
}

.app-swal-actions {
    margin: 1.5rem 0 0 0 !important;
    padding: 0 !important;
    gap: 0.5rem !important;
    justify-content: flex-end !important;
    width: 100% !important;
}

/* ─── TOMBOL DIALOG ─────────────────────────────────────────────────────────── */

/* Confirm — biru */
.app-swal-confirm {
    display: inline-flex !important;
    align-items: center !important;
    justify-content: center !important;
    padding: 0.5rem 1.125rem !important;
    background-color: #2563EB !important;
    color: #fff !important;
    font-family: 'Inter', ui-sans-serif, sans-serif !important;
    font-size: 0.875rem !important;
    font-weight: 500 !important;
    border-radius: 0.5rem !important;
    border: none !important;
    cursor: pointer !important;
    transition: background-color 0.15s ease, box-shadow 0.15s ease !important;
    box-shadow: 0 1px 3px rgba(37, 99, 235, 0.35) !important;
    outline: none !important;
}
.app-swal-confirm:hover { background-color: #1D4ED8 !important; }
.app-swal-confirm:focus {
    box-shadow: 0 0 0 3px rgba(147, 197, 253, 0.6) !important;
}

/* Confirm — merah (untuk hapus) */
.app-swal-danger {
    display: inline-flex !important;
    align-items: center !important;
    justify-content: center !important;
    padding: 0.5rem 1.125rem !important;
    background-color: #DC2626 !important;
    color: #fff !important;
    font-family: 'Inter', ui-sans-serif, sans-serif !important;
    font-size: 0.875rem !important;
    font-weight: 500 !important;
    border-radius: 0.5rem !important;
    border: none !important;
    cursor: pointer !important;
    transition: background-color 0.15s ease !important;
    box-shadow: 0 1px 3px rgba(220, 38, 38, 0.35) !important;
    outline: none !important;
}
.app-swal-danger:hover { background-color: #B91C1C !important; }
.app-swal-danger:focus {
    box-shadow: 0 0 0 3px rgba(252, 165, 165, 0.6) !important;
}

/* Cancel — abu-abu */
.app-swal-cancel {
    display: inline-flex !important;
    align-items: center !important;
    justify-content: center !important;
    padding: 0.5rem 1.125rem !important;
    background-color: #fff !important;
    color: #374151 !important;
    font-family: 'Inter', ui-sans-serif, sans-serif !important;
    font-size: 0.875rem !important;
    font-weight: 500 !important;
    border-radius: 0.5rem !important;
    border: 1px solid #D1D5DB !important;
    cursor: pointer !important;
    transition: background-color 0.15s ease !important;
    outline: none !important;
}
.app-swal-cancel:hover { background-color: #F9FAFB !important; }
.app-swal-cancel:focus {
    box-shadow: 0 0 0 3px rgba(209, 213, 219, 0.7) !important;
}

/* ─── ANIMASI DIALOG ─────────────────────────────────────────────────────────── */

.app-swal-enter {
    animation: swalEnter 0.22s cubic-bezier(0.34, 1.28, 0.64, 1) forwards !important;
}
.app-swal-leave {
    animation: swalLeave 0.15s ease-in forwards !important;
}

@keyframes swalEnter {
    from { opacity: 0; transform: translateY(-14px) scale(0.96); }
    to   { opacity: 1; transform: translateY(0)     scale(1); }
}
@keyframes swalLeave {
    from { opacity: 1; transform: scale(1); }
    to   { opacity: 0; transform: scale(0.97); }
}

/* ─── TOAST NOTIFICATION ─────────────────────────────────────────────────────── */
/*
   Desain khas: dark background (zinc-900), slide dari kanan,
   progress bar gradien biru, border transparan.
   Sangat berbeda dari default SweetAlert2 toast.
*/

.app-toast {
    font-family: 'Inter', ui-sans-serif, system-ui, sans-serif !important;
    background: #18181B !important;              /* zinc-900 */
    border: 1px solid rgba(255, 255, 255, 0.07) !important;
    border-radius: 0.875rem !important;          /* 14px */
    padding: 0.875rem 1.125rem !important;
    box-shadow:
        0 10px 32px rgba(0, 0, 0, 0.28),
        0  0    0  1px rgba(0, 0, 0, 0.1) !important;
    min-width: 18rem !important;
    max-width: 22rem !important;
}

.app-toast-title {
    font-size: 0.875rem !important;
    font-weight: 500 !important;
    color: #F4F4F5 !important;                   /* zinc-100 */
    text-align: left !important;
    margin: 0 !important;
    padding: 0 !important;
    line-height: 1.4 !important;
}

/* Icon kecil untuk toast */
.app-toast .swal2-icon {
    width: 1.25rem !important;
    height: 1.25rem !important;
    min-width: 1.25rem !important;
    margin: 0 0.625rem 0 0 !important;
    border-width: 1.5px !important;
    font-size: 0.5rem !important;
}

/* Warna icon di atas dark background */
.app-toast .swal2-icon.swal2-success {
    border-color: #34D399 !important;
    color: #34D399 !important;
}
.app-toast .swal2-icon.swal2-success .swal2-success-ring {
    border-color: rgba(52, 211, 153, 0.25) !important;
}
.app-toast .swal2-icon.swal2-success [class^='swal2-success-line'] {
    background-color: #34D399 !important;
}
.app-toast .swal2-icon.swal2-error {
    border-color: #F87171 !important;
    color: #F87171 !important;
}
.app-toast .swal2-icon.swal2-error [class^='swal2-x-mark-line'] {
    background-color: #F87171 !important;
}
.app-toast .swal2-icon.swal2-warning {
    border-color: #FBBF24 !important;
    color: #FBBF24 !important;
}
.app-toast .swal2-icon.swal2-info {
    border-color: #60A5FA !important;
    color: #60A5FA !important;
}

/* Progress bar — gradien biru tipis */
.app-toast-bar {
    background: linear-gradient(90deg, #3B82F6, #818CF8) !important;
    height: 2px !important;
    border-radius: 0 0 0.875rem 0.875rem !important;
    bottom: 0 !important;
    left: 0 !important;
    opacity: 0.85 !important;
}

/* ─── ANIMASI TOAST ─────────────────────────────────────────────────────────── */

.app-toast-enter {
    animation: toastIn 0.28s cubic-bezier(0.16, 1, 0.3, 1) forwards !important;
}
.app-toast-leave {
    animation: toastOut 0.2s ease-in forwards !important;
}

@keyframes toastIn {
    from { opacity: 0; transform: translateX(56px) scale(0.95); }
    to   { opacity: 1; transform: translateX(0)    scale(1); }
}
@keyframes toastOut {
    from { opacity: 1; transform: translateX(0)    scale(1); }
    to   { opacity: 0; transform: translateX(28px) scale(0.97); }
}
```

---

## File 3: Update `resources/js/app.js`

```js
import './bootstrap';
import Alpine from 'alpinejs';
import './swal';    // ← import config SweetAlert2 (expose ke window)

window.Alpine = Alpine;

// Dark mode store
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

## File 4: Update `resources/css/app.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* SweetAlert2 custom theme */
@import './sweetalert.css';
```

---

## File 5: Update `vite.config.js`

Tidak ada perubahan — sweetalert.css sudah di-import dari app.css, dan sweetalert2 JS di-import dari swal.js.

```bash
# Install ulang dependencies setelah update
npm install
npm run dev
```

---

## File 6: Update Layout — Flash Message Data Holder

Di `resources/views/layouts/app.blade.php`, tambahkan hidden div setelah `<body>`:

```blade
{{-- Flash messages — dibaca oleh swal.js untuk auto-fire toast --}}
<div id="flash-messages"
     data-success="{{ session('success') }}"
     data-error="{{ session('error') }}"
     data-warning="{{ session('warning') }}"
     data-info="{{ session('info') }}"
     hidden>
</div>
```

Hapus `<x-alert />` dari layout — digantikan oleh SweetAlert2 toast otomatis.

---

## Panduan Penggunaan

### 1. Toast Notifikasi (auto dari session)

Tidak perlu kode Blade tambahan. Cukup flash session di controller:

```php
// Controller — setelah operasi berhasil
return redirect()->route('products.index')
    ->with('success', 'Produk berhasil ditambahkan.');

// Error
return redirect()->back()
    ->with('error', 'Terjadi kesalahan. Coba lagi.');

// Warning
return redirect()->route('products.index')
    ->with('warning', 'Stok produk hampir habis.');

// Info
return redirect()->route('products.index')
    ->with('info', 'Data sedang diproses.');
```

Toast akan muncul otomatis saat halaman load dengan animasi slide-kanan + dark background.

### 2. Tombol Hapus di Tabel

```blade
{{-- Gantikan form DELETE dengan button + onclick --}}
<button
    type="button"
    onclick="confirmDelete(
        '{{ route('products.destroy', $product->id) }}',
        '{{ addslashes($product->name) }}'
    )"
    class="text-xs font-medium text-red-600 hover:text-red-800 transition-colors">
    Hapus
</button>
```

Dialog akan muncul:
- Layout kustom (ikon + nama item + teks peringatan)
- Tombol "Ya, hapus" (merah) + "Batal" (abu-abu)
- Jika konfirm: submit form DELETE via JS
- Animasi slide-down halus

### 3. Konfirmasi Aksi Lain (bukan hapus)

```blade
{{-- Tombol aktivasi / perubahan status --}}
<button
    type="button"
    onclick="confirmAction(
        {
            title: 'Nonaktifkan produk ini?',
            description: 'Produk tidak akan terlihat oleh pelanggan.',
            confirmText: 'Ya, nonaktifkan',
            icon: 'yellow'
        },
        '{{ route('products.deactivate', $product->id) }}',
        'POST'
    )"
    class="text-xs font-medium text-amber-600 hover:text-amber-800">
    Nonaktifkan
</button>
```

### 4. Toast Manual dari JavaScript

Jika perlu trigger toast dari JS (bukan session flash):

```js
// Sukses
Toast.fire({ icon: 'success', title: 'Data berhasil disimpan' });

// Error
Toast.fire({ icon: 'error', title: 'Gagal menyimpan data' });

// Warning
Toast.fire({ icon: 'warning', title: 'Koneksi tidak stabil' });

// Info
Toast.fire({ icon: 'info', title: 'Sedang memproses...' });
```

### 5. Dialog Custom (Advanced)

```js
// Dialog dengan loading state setelah konfirmasi
const result = await Confirm.fire({
    title: 'Ekspor data?',
    html: `
        <div class="text-left">
            <p class="text-sm text-gray-500">
                Semua <strong class="text-gray-700">1.284 produk</strong>
                akan diekspor ke file Excel.
            </p>
        </div>
    `,
    confirmButtonText: 'Ya, ekspor',
});

if (result.isConfirmed) {
    Swal.fire({
        title: 'Mengekspor data...',
        html: '<p class="text-sm text-gray-500">Mohon tunggu sebentar.</p>',
        allowOutsideClick: false,
        showConfirmButton: false,
        didOpen: () => Swal.showLoading(),
    });

    // Lakukan proses...
}
```

### 6. Validasi Error dari AJAX

```js
// Jika menggunakan fetch/axios untuk submit form:
try {
    const response = await fetch(url, { method: 'POST', body: formData });
    const data = await response.json();

    if (data.success) {
        Toast.fire({ icon: 'success', title: data.message });
    } else {
        Toast.fire({ icon: 'error', title: data.message ?? 'Terjadi kesalahan' });
    }
} catch (e) {
    Toast.fire({ icon: 'error', title: 'Koneksi gagal. Coba lagi.' });
}
```

---

## Checklist Implementasi

Wajib diverifikasi saat `new-project`:

- [ ] `npm install sweetalert2` sudah dijalankan
- [ ] `resources/js/swal.js` sudah dibuat
- [ ] `resources/css/sweetalert.css` sudah dibuat
- [ ] `resources/css/app.css` sudah import `./sweetalert.css`
- [ ] `resources/js/app.js` sudah import `./swal`
- [ ] Layout `app.blade.php` punya `<div id="flash-messages" ...>` setelah `<body>`
- [ ] Tidak ada lagi `<x-alert />` di layout (digantikan toast)
- [ ] `npm run build` berhasil tanpa error

Wajib diverifikasi saat `new-module`:

- [ ] Tombol hapus di tabel menggunakan `onclick="confirmDelete(...)"` — TIDAK ADA form inline
- [ ] Tidak ada `onsubmit="return confirm(...)"` — semua konfirmasi pakai SweetAlert2
- [ ] Redirect controller menggunakan `->with('success', '...')` atau `->with('error', '...')`
- [ ] Tidak ada `<x-modal>` untuk tujuan konfirmasi hapus (sudah digantikan)
