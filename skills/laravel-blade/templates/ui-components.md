# Template UI Components: Blade Component Library

Semua komponen ini dibuat saat `new-project`. Simpan di `resources/views/components/`.

---

## Layout: `resources/views/layouts/app.blade.php`

Layout utama dengan sidebar fixed + sticky navbar. Mendukung mobile responsive via Alpine.js.

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}" class="h-full">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ $title ?? config('app.name') }} — {{ config('app.name') }}</title>

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">

    @stack('styles')
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="h-full bg-gray-50 font-['Inter']" x-data="{ sidebarOpen: false }">

{{-- Mobile Overlay --}}
<div x-show="sidebarOpen"
     x-transition:enter="transition-opacity ease-linear duration-200"
     x-transition:enter-start="opacity-0"
     x-transition:enter-end="opacity-100"
     x-transition:leave="transition-opacity ease-linear duration-200"
     x-transition:leave-start="opacity-100"
     x-transition:leave-end="opacity-0"
     @click="sidebarOpen = false"
     class="fixed inset-0 z-20 bg-gray-900/50 lg:hidden"
     style="display:none">
</div>

{{-- Sidebar --}}
<aside class="fixed inset-y-0 left-0 z-30 w-64 bg-white border-r border-gray-200 flex flex-col
              transform transition-transform duration-200 ease-in-out
              -translate-x-full lg:translate-x-0"
       :class="sidebarOpen ? 'translate-x-0' : '-translate-x-full lg:translate-x-0'">

    {{-- Logo --}}
    <div class="flex items-center h-16 px-6 border-b border-gray-200 flex-shrink-0">
        <a href="{{ route('dashboard') }}" class="flex items-center gap-2">
            <div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center">
                <span class="text-white font-bold text-sm">{{ substr(config('app.name'), 0, 1) }}</span>
            </div>
            <span class="text-gray-900 font-semibold text-base">{{ config('app.name') }}</span>
        </a>
    </div>

    {{-- Navigation --}}
    <nav class="flex-1 overflow-y-auto px-3 py-4">
        @include('layouts._partials.sidebar')
    </nav>

    {{-- User Profile --}}
    <div class="flex-shrink-0 border-t border-gray-200 p-3">
        <div class="flex items-center gap-3 px-3 py-2 rounded-lg hover:bg-gray-50 cursor-pointer"
             x-data="{ open: false }" @click="open = !open" class="relative">
            <div class="w-8 h-8 rounded-full bg-blue-600 flex items-center justify-center flex-shrink-0">
                <span class="text-white text-sm font-medium">
                    {{ strtoupper(substr(auth()->user()->name ?? 'U', 0, 1)) }}
                </span>
            </div>
            <div class="flex-1 min-w-0">
                <p class="text-sm font-medium text-gray-900 truncate">{{ auth()->user()->name ?? '' }}</p>
                <p class="text-xs text-gray-500 truncate">{{ auth()->user()->email ?? '' }}</p>
            </div>
            {{-- Dropdown user --}}
            <div x-show="open" @click.outside="open = false"
                 x-transition:enter="transition ease-out duration-100"
                 x-transition:enter-start="opacity-0 scale-95"
                 x-transition:enter-end="opacity-100 scale-100"
                 class="absolute bottom-full left-0 right-0 mb-2 bg-white rounded-xl shadow-lg
                        border border-gray-200 overflow-hidden z-10"
                 style="display:none">
                <a href="{{ route('profile.edit') }}" class="flex items-center gap-2 px-4 py-2.5 text-sm text-gray-700 hover:bg-gray-50">
                    Profil Saya
                </a>
                <hr class="border-gray-200">
                <form method="POST" action="{{ route('logout') }}">
                    @csrf
                    <button type="submit" class="w-full flex items-center gap-2 px-4 py-2.5 text-sm text-red-600 hover:bg-red-50">
                        Keluar
                    </button>
                </form>
            </div>
        </div>
    </div>
</aside>

{{-- Main Content --}}
<div class="lg:pl-64 flex flex-col min-h-full">

    {{-- Sticky Navbar --}}
    <header class="sticky top-0 z-20 bg-white border-b border-gray-200 h-16 flex items-center px-4 sm:px-6 gap-4">
        {{-- Hamburger (mobile) --}}
        <button @click="sidebarOpen = !sidebarOpen"
                class="lg:hidden p-2 rounded-lg text-gray-500 hover:bg-gray-100">
            <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M3.75 6.75h16.5M3.75 12h16.5m-16.5 5.25h16.5"/>
            </svg>
        </button>

        {{-- Breadcrumb --}}
        <div class="flex-1">
            @isset($breadcrumbs)
                <x-breadcrumb :items="$breadcrumbs" />
            @endisset
        </div>

        {{-- Navbar Actions --}}
        <div class="flex items-center gap-2">
            {{-- Notifications --}}
            <button class="relative p-2 rounded-lg text-gray-500 hover:bg-gray-100">
                <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M14.857 17.082a23.848 23.848 0 0 0 5.454-1.31A8.967 8.967 0 0 1 18 9.75V9A6 6 0 0 0 6 9v.75a8.967 8.967 0 0 1-2.312 6.022c1.733.64 3.56 1.085 5.455 1.31m5.714 0a24.255 24.255 0 0 1-5.714 0m5.714 0a3 3 0 1 1-5.714 0"/>
                </svg>
            </button>
        </div>
    </header>

    {{-- Page Content --}}
    <main class="flex-1 p-4 sm:p-6">
        {{-- Flash Messages — dibaca oleh swal.js untuk auto-fire toast --}}
        <div id="flash-messages"
             data-success="{{ session('success') }}"
             data-error="{{ session('error') }}"
             data-warning="{{ session('warning') }}"
             data-info="{{ session('info') }}"
             hidden>
        </div>

        {{-- Page Header (title + actions) --}}
        @isset($header)
            <div class="mb-6">
                {{ $header }}
            </div>
        @endisset

        {{-- Main Slot --}}
        {{ $slot }}
    </main>
</div>

@stack('scripts')
</body>
</html>
```

---

## Sidebar: `resources/views/layouts/_partials/sidebar.blade.php`

Tambahkan item menu di sini setiap kali `new-module` dijalankan.

```blade
{{-- Dashboard --}}
<a href="{{ route('dashboard') }}"
   class="flex items-center gap-3 px-3 py-2 text-sm rounded-lg transition-colors duration-150
          {{ request()->routeIs('dashboard') ? 'bg-blue-50 text-blue-700 font-semibold' : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900' }}">
    <svg class="w-5 h-5 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" d="m2.25 12 8.954-8.955c.44-.439 1.152-.439 1.591 0L21.75 12M4.5 9.75v10.125c0 .621.504 1.125 1.125 1.125H9.75v-4.875c0-.621.504-1.125 1.125-1.125h2.25c.621 0 1.125.504 1.125 1.125V21h4.125c.621 0 1.125-.504 1.125-1.125V9.75M8.25 21h8.25"/>
    </svg>
    Dashboard
</a>

{{--
    === MODUL NAVIGATION ===
    Setiap new-module menambahkan item di sini.
    Format:

    <p class="px-3 pt-5 pb-1 text-xs font-semibold text-gray-400 uppercase tracking-wider">
        Nama Section
    </p>
    <a href="{{ route('products.index') }}"
       class="flex items-center gap-3 px-3 py-2 text-sm rounded-lg transition-colors duration-150
              {{ request()->routeIs('products.*') ? 'bg-blue-50 text-blue-700 font-semibold' : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900' }}">
        <svg class="w-5 h-5 flex-shrink-0">...</svg>
        Produk
    </a>
    === AKHIR MODUL NAVIGATION ===
--}}

{{-- System Section --}}
<p class="px-3 pt-5 pb-1 text-xs font-semibold text-gray-400 uppercase tracking-wider">Sistem</p>
<a href="{{ route('profile.edit') }}"
   class="flex items-center gap-3 px-3 py-2 text-sm rounded-lg transition-colors duration-150
          {{ request()->routeIs('profile.*') ? 'bg-blue-50 text-blue-700 font-semibold' : 'text-gray-600 hover:bg-gray-100 hover:text-gray-900' }}">
    <svg class="w-5 h-5 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" d="M17.982 18.725A7.488 7.488 0 0 0 12 15.75a7.488 7.488 0 0 0-5.982 2.975m11.963 0a9 9 0 1 0-11.963 0m11.963 0A8.966 8.966 0 0 1 12 21a8.966 8.966 0 0 1-5.982-2.275M15 9.75a3 3 0 1 1-6 0 3 3 0 0 1 6 0Z"/>
    </svg>
    Profil
</a>
```

---

## Alert: `resources/views/components/alert.blade.php`

> **Catatan Penggunaan:** Komponen ini untuk **inline alert** yang di-pass manual (`<x-alert type="info" message="Pesan..." />`).
> Flash session dari controller (success/error/warning/info via `->with()`) **TIDAK BOLEH** menggunakan komponen ini
> — sudah ditangani otomatis oleh SweetAlert2 toast via `<div id="flash-messages">` di layout.

Menampilkan flash session secara otomatis, bisa juga di-pass manual.

```blade
@props(['type' => null, 'message' => null, 'dismissible' => true])

@php
    $map = [
        'success' => ['bg'=>'bg-emerald-50','border'=>'border-emerald-300','text'=>'text-emerald-800','icon_color'=>'text-emerald-500'],
        'error'   => ['bg'=>'bg-red-50',    'border'=>'border-red-300',    'text'=>'text-red-800',    'icon_color'=>'text-red-500'],
        'warning' => ['bg'=>'bg-amber-50',  'border'=>'border-amber-300',  'text'=>'text-amber-800',  'icon_color'=>'text-amber-500'],
        'info'    => ['bg'=>'bg-sky-50',    'border'=>'border-sky-300',    'text'=>'text-sky-800',    'icon_color'=>'text-sky-500'],
    ];

    // Auto-detect dari session jika tidak di-pass manual
    if (! $type) {
        if (session('success')) { $type = 'success'; $message = session('success'); }
        elseif (session('error')) { $type = 'error'; $message = session('error'); }
        elseif (session('warning')) { $type = 'warning'; $message = session('warning'); }
        elseif (session('info'))  { $type = 'info';  $message = session('info'); }
    }

    $style = $map[$type] ?? null;
@endphp

@if ($style && $message)
<div x-data="{ show: true }" x-show="show"
     x-transition:leave="transition ease-in duration-200"
     x-transition:leave-start="opacity-100 translate-y-0"
     x-transition:leave-end="opacity-0 -translate-y-1"
     class="mb-5 flex items-start gap-3 p-4 rounded-xl border {{ $style['bg'] }} {{ $style['border'] }} {{ $style['text'] }}"
     role="alert">

    {{-- Icon --}}
    <div class="flex-shrink-0 {{ $style['icon_color'] }}">
        @if($type === 'success')
            <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 1 0 0-16 8 8 0 0 0 0 16Zm3.857-9.809a.75.75 0 0 0-1.214-.882l-3.483 4.79-1.88-1.88a.75.75 0 1 0-1.06 1.061l2.5 2.5a.75.75 0 0 0 1.137-.089l4-5.5Z" clip-rule="evenodd"/>
            </svg>
        @elseif($type === 'error')
            <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 1 0 0-16 8 8 0 0 0 0 16ZM8.28 7.22a.75.75 0 0 0-1.06 1.06L8.94 10l-1.72 1.72a.75.75 0 1 0 1.06 1.06L10 11.06l1.72 1.72a.75.75 0 1 0 1.06-1.06L11.06 10l1.72-1.72a.75.75 0 0 0-1.06-1.06L10 8.94 8.28 7.22Z" clip-rule="evenodd"/>
            </svg>
        @elseif($type === 'warning')
            <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M8.485 2.495c.673-1.167 2.357-1.167 3.03 0l6.28 10.875c.673 1.167-.17 2.625-1.516 2.625H3.72c-1.347 0-2.189-1.458-1.515-2.625L8.485 2.495ZM10 5a.75.75 0 0 1 .75.75v3.5a.75.75 0 0 1-1.5 0v-3.5A.75.75 0 0 1 10 5Zm0 9a1 1 0 1 0 0-2 1 1 0 0 0 0 2Z" clip-rule="evenodd"/>
            </svg>
        @else
            <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M18 10a8 8 0 1 1-16 0 8 8 0 0 1 16 0Zm-7-4a1 1 0 1 1-2 0 1 1 0 0 1 2 0ZM9 9a.75.75 0 0 0 0 1.5h.253a.25.25 0 0 1 .244.304l-.459 2.066A1.75 1.75 0 0 0 10.747 15H11a.75.75 0 0 0 0-1.5h-.253a.25.25 0 0 1-.244-.304l.459-2.066A1.75 1.75 0 0 0 9.253 9H9Z" clip-rule="evenodd"/>
            </svg>
        @endif
    </div>

    {{-- Message --}}
    <p class="flex-1 text-sm font-medium">{{ $message }}</p>

    {{-- Dismiss --}}
    @if($dismissible)
        <button @click="show = false" class="flex-shrink-0 opacity-60 hover:opacity-100 transition-opacity">
            <svg class="w-4 h-4" viewBox="0 0 20 20" fill="currentColor">
                <path d="M6.28 5.22a.75.75 0 0 0-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 1 0 1.06 1.06L10 11.06l3.72 3.72a.75.75 0 1 0 1.06-1.06L11.06 10l3.72-3.72a.75.75 0 0 0-1.06-1.06L10 8.94 6.28 5.22Z"/>
            </svg>
        </button>
    @endif
</div>
@endif
```

---

## Card: `resources/views/components/card.blade.php`

```blade
@props(['title' => null, 'padding' => true])

<div {{ $attributes->merge(['class' => 'bg-white rounded-xl border border-gray-200 shadow-sm overflow-hidden']) }}>

    @if ($title || isset($actions))
        <div class="flex items-center justify-between px-6 py-4 border-b border-gray-200">
            @if ($title)
                <h3 class="text-base font-semibold text-gray-900">{{ $title }}</h3>
            @endif
            @isset($actions)
                <div class="flex items-center gap-2">
                    {{ $actions }}
                </div>
            @endisset
        </div>
    @endif

    <div class="{{ $padding ? 'p-6' : '' }}">
        {{ $slot }}
    </div>

    @isset($footer)
        <div class="px-6 py-4 bg-gray-50 border-t border-gray-200">
            {{ $footer }}
        </div>
    @endisset
</div>
```

---

## Stat Card: `resources/views/components/stat-card.blade.php`

Untuk dashboard metrics.

```blade
@props([
    'label'  => 'Label',
    'value'  => '0',
    'change' => null,       // mis. '+12% dari bulan lalu'
    'trend'  => 'up',       // 'up' | 'down' | 'neutral'
    'color'  => 'blue',     // 'blue' | 'emerald' | 'red' | 'amber' | 'violet'
])

@php
    $colors = [
        'blue'    => ['bg' => 'bg-blue-50',    'icon' => 'text-blue-600'],
        'emerald' => ['bg' => 'bg-emerald-50', 'icon' => 'text-emerald-600'],
        'red'     => ['bg' => 'bg-red-50',     'icon' => 'text-red-600'],
        'amber'   => ['bg' => 'bg-amber-50',   'icon' => 'text-amber-600'],
        'violet'  => ['bg' => 'bg-violet-50',  'icon' => 'text-violet-600'],
    ];
    $c = $colors[$color] ?? $colors['blue'];
    $trendColor = $trend === 'up' ? 'text-emerald-600' : ($trend === 'down' ? 'text-red-600' : 'text-gray-500');
@endphp

<div class="bg-white rounded-xl border border-gray-200 shadow-sm p-6">
    <div class="flex items-start justify-between">
        <div class="flex-1 min-w-0">
            <p class="text-sm text-gray-500 truncate">{{ $label }}</p>
            <p class="text-2xl font-bold text-gray-900 mt-1">{{ $value }}</p>
            @if ($change)
                <p class="text-xs {{ $trendColor }} mt-1 flex items-center gap-1">
                    @if ($trend === 'up')
                        <svg class="w-3 h-3" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M10 17a.75.75 0 0 1-.75-.75V5.612L5.29 9.77a.75.75 0 0 1-1.08-1.04l5.25-5.5a.75.75 0 0 1 1.08 0l5.25 5.5a.75.75 0 1 1-1.08 1.04l-3.96-4.158V16.25A.75.75 0 0 1 10 17Z" clip-rule="evenodd"/>
                        </svg>
                    @elseif ($trend === 'down')
                        <svg class="w-3 h-3" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M10 3a.75.75 0 0 1 .75.75v10.638l3.96-4.158a.75.75 0 1 1 1.08 1.04l-5.25 5.5a.75.75 0 0 1-1.08 0l-5.25-5.5a.75.75 0 1 1 1.08-1.04l3.96 4.158V3.75A.75.75 0 0 1 10 3Z" clip-rule="evenodd"/>
                        </svg>
                    @endif
                    {{ $change }}
                </p>
            @endif
        </div>
        @isset($icon)
            <div class="flex-shrink-0 w-12 h-12 {{ $c['bg'] }} {{ $c['icon'] }} rounded-xl flex items-center justify-center ml-4">
                {{ $icon }}
            </div>
        @endisset
    </div>
</div>
```

**Penggunaan:**
```blade
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6 mb-6">
    <x-stat-card
        label="Total Produk"
        value="1.284"
        change="+12% dari bulan lalu"
        trend="up"
        color="blue">
        <x-slot name="icon">
            <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" d="M21 7.5l-9-5.25L3 7.5m18 0l-9 5.25m9-5.25v9l-9 5.25M3 7.5l9 5.25M3 7.5v9l9 5.25m0-9v9"/>
            </svg>
        </x-slot>
    </x-stat-card>
</div>
```

---

## Badge: `resources/views/components/badge.blade.php`

```blade
@props(['type' => 'default', 'dot' => false])

@php
    $styles = [
        'success' => 'bg-emerald-50 text-emerald-700 ring-emerald-600/20',
        'danger'  => 'bg-red-50 text-red-700 ring-red-600/20',
        'warning' => 'bg-amber-50 text-amber-700 ring-amber-600/20',
        'info'    => 'bg-sky-50 text-sky-700 ring-sky-600/20',
        'default' => 'bg-gray-100 text-gray-600 ring-gray-500/20',
    ];
    $dotColors = [
        'success' => 'bg-emerald-500',
        'danger'  => 'bg-red-500',
        'warning' => 'bg-amber-500',
        'info'    => 'bg-sky-500',
        'default' => 'bg-gray-400',
    ];
    $style = $styles[$type] ?? $styles['default'];
    $dotColor = $dotColors[$type] ?? $dotColors['default'];
@endphp

<span {{ $attributes->merge(['class' => "inline-flex items-center gap-1.5 px-2.5 py-0.5 text-xs font-medium rounded-full ring-1 ring-inset {$style}"]) }}>
    @if ($dot)
        <svg class="h-1.5 w-1.5 {{ $dotColor }}" viewBox="0 0 6 6" aria-hidden="true">
            <circle cx="3" cy="3" r="3"/>
        </svg>
    @endif
    {{ $slot }}
</span>
```

**Penggunaan:**
```blade
<x-badge type="success" :dot="true">Aktif</x-badge>
<x-badge type="danger" :dot="true">Nonaktif</x-badge>
<x-badge type="warning">Pending</x-badge>
<x-badge type="info">Draft</x-badge>
```

---

## Button: `resources/views/components/button.blade.php`

```blade
@props([
    'variant' => 'primary',  // primary | secondary | danger | ghost | warning
    'size'    => 'md',        // sm | md | lg
    'href'    => null,
    'type'    => 'button',
    'loading' => false,
])

@php
    $variants = [
        'primary'   => 'bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500 shadow-sm',
        'secondary' => 'bg-white text-gray-700 hover:bg-gray-50 border border-gray-300 focus:ring-gray-300 shadow-sm',
        'danger'    => 'bg-red-600 text-white hover:bg-red-700 focus:ring-red-500 shadow-sm',
        'ghost'     => 'text-gray-600 hover:bg-gray-100 hover:text-gray-900 focus:ring-gray-300',
        'warning'   => 'bg-amber-500 text-white hover:bg-amber-600 focus:ring-amber-500 shadow-sm',
    ];
    $sizes = [
        'sm' => 'px-3 py-1.5 text-xs',
        'md' => 'px-4 py-2 text-sm',
        'lg' => 'px-5 py-2.5 text-base',
    ];
    $base = 'inline-flex items-center justify-center gap-2 font-medium rounded-lg
             focus:outline-none focus:ring-2 focus:ring-offset-2
             transition-colors duration-150 disabled:opacity-60 disabled:cursor-not-allowed';
    $classes = "{$base} {$variants[$variant]} {$sizes[$size]}";
@endphp

@if ($href)
    <a href="{{ $href }}" {{ $attributes->merge(['class' => $classes]) }}>
        {{ $slot }}
    </a>
@else
    <button type="{{ $type }}" {{ $attributes->merge(['class' => $classes]) }} @disabled($loading)>
        @if ($loading)
            <svg class="animate-spin w-4 h-4" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"/>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 0 1 8-8V0C5.373 0 0 5.373 0 12h4z"/>
            </svg>
        @endif
        {{ $slot }}
    </button>
@endif
```

---

## Modal: `resources/views/components/modal.blade.php`

```blade
@props(['id', 'title', 'maxWidth' => 'md'])

@php
    $widths = ['sm' => 'max-w-sm', 'md' => 'max-w-md', 'lg' => 'max-w-lg', 'xl' => 'max-w-xl', '2xl' => 'max-w-2xl'];
    $maxWidthClass = $widths[$maxWidth] ?? $widths['md'];
@endphp

<div x-data="{ open: false }"
     x-on:open-modal.window="$event.detail.id === '{{ $id }}' ? open = true : null"
     x-on:close-modal.window="open = false"
     x-on:keydown.escape.window="open = false">

    {{-- Trigger slot --}}
    @isset($trigger)
        <div @click="open = true">{{ $trigger }}</div>
    @endisset

    {{-- Overlay --}}
    <div x-show="open"
         x-transition:enter="ease-out duration-200"
         x-transition:enter-start="opacity-0"
         x-transition:enter-end="opacity-100"
         x-transition:leave="ease-in duration-150"
         x-transition:leave-start="opacity-100"
         x-transition:leave-end="opacity-0"
         @click.self="open = false"
         class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-gray-900/60 backdrop-blur-sm"
         style="display:none">

        {{-- Dialog --}}
        <div x-show="open"
             x-transition:enter="ease-out duration-200"
             x-transition:enter-start="opacity-0 scale-95"
             x-transition:enter-end="opacity-100 scale-100"
             x-transition:leave="ease-in duration-150"
             x-transition:leave-start="opacity-100 scale-100"
             x-transition:leave-end="opacity-0 scale-95"
             class="w-full {{ $maxWidthClass }} bg-white rounded-2xl shadow-xl overflow-hidden">

            {{-- Header --}}
            <div class="flex items-center justify-between px-6 py-4 border-b border-gray-200">
                <h3 class="text-base font-semibold text-gray-900">{{ $title }}</h3>
                <button @click="open = false" class="p-1 rounded-lg text-gray-400 hover:bg-gray-100 hover:text-gray-600">
                    <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                        <path d="M6.28 5.22a.75.75 0 0 0-1.06 1.06L8.94 10l-3.72 3.72a.75.75 0 1 0 1.06 1.06L10 11.06l3.72 3.72a.75.75 0 1 0 1.06-1.06L11.06 10l3.72-3.72a.75.75 0 0 0-1.06-1.06L10 8.94 6.28 5.22Z"/>
                    </svg>
                </button>
            </div>

            {{-- Body --}}
            <div class="p-6">{{ $slot }}</div>

            {{-- Footer --}}
            @isset($footer)
                <div class="px-6 py-4 bg-gray-50 border-t border-gray-200 flex justify-end gap-3">
                    {{ $footer }}
                </div>
            @endisset
        </div>
    </div>
</div>
```

**Penggunaan — Non-konfirmasi (form, detail, dsb.):**
```blade
{{-- Trigger modal via event --}}
<x-button variant="secondary" onclick="$dispatch('open-modal', { id: 'edit-notes' })">
    Edit Catatan
</x-button>

<x-modal id="edit-notes" title="Edit Catatan">
    <form method="POST" action="{{ route('notes.update', $note->id) }}">
        @csrf @method('PUT')
        <x-form.textarea name="content" label="Catatan" :required="true" />
        <x-slot name="footer">
            <x-button variant="secondary" @click="open = false">Batal</x-button>
            <x-button variant="primary" type="submit">Simpan</x-button>
        </x-slot>
    </form>
</x-modal>
```

> **Penting:** `<x-modal>` TIDAK BOLEH digunakan untuk konfirmasi hapus.
> Hapus data WAJIB menggunakan `confirmDelete()` dari SweetAlert2:
>
> ```blade
> <button
>     type="button"
>     onclick="confirmDelete(
>         '{{ route('products.destroy', $product->id) }}',
>         '{{ addslashes($product->name) }}'
>     )"
>     class="text-xs font-medium text-red-600 hover:text-red-800">
>     Hapus
> </button>
> ```

---

## Empty State: `resources/views/components/empty-state.blade.php`

```blade
@props(['title' => 'Tidak ada data', 'description' => null])

<div class="text-center py-16 px-6">
    <div class="w-16 h-16 bg-gray-100 rounded-2xl flex items-center justify-center mx-auto mb-4">
        <svg class="w-8 h-8 text-gray-400" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" d="M2.25 13.5h3.86a2.25 2.25 0 0 1 2.012 1.244l.256.512a2.25 2.25 0 0 0 2.013 1.244h3.218a2.25 2.25 0 0 0 2.013-1.244l.256-.512a2.25 2.25 0 0 1 2.013-1.244h3.859m-19.5.338V18a2.25 2.25 0 0 0 2.25 2.25h15A2.25 2.25 0 0 0 21.75 18v-4.162c0-.224-.034-.447-.1-.661L19.24 5.338a2.25 2.25 0 0 0-2.15-1.588H6.911a2.25 2.25 0 0 0-2.15 1.588L2.35 13.177a2.25 2.25 0 0 0-.1.661Z"/>
        </svg>
    </div>
    <h3 class="text-base font-semibold text-gray-900">{{ $title }}</h3>
    @if ($description)
        <p class="text-sm text-gray-500 mt-1">{{ $description }}</p>
    @endif
    @isset($action)
        <div class="mt-4">{{ $action }}</div>
    @endisset
</div>
```

---

## Breadcrumb: `resources/views/components/breadcrumb.blade.php`

```blade
@props(['items' => []])

{{-- items: [['label' => 'Produk', 'url' => route('products.index')], ['label' => 'Edit']] --}}
<nav class="flex items-center gap-1 text-sm">
    <a href="{{ route('dashboard') }}" class="text-gray-400 hover:text-gray-600 transition-colors">
        <svg class="w-4 h-4" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M9.293 2.293a1 1 0 0 1 1.414 0l7 7A1 1 0 0 1 17 11h-1v6a1 1 0 0 1-1 1h-2a1 1 0 0 1-1-1v-3a1 1 0 0 0-1-1H9a1 1 0 0 0-1 1v3a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1v-6H3a1 1 0 0 1-.707-1.707l7-7Z" clip-rule="evenodd"/>
        </svg>
    </a>

    @foreach ($items as $item)
        <svg class="w-4 h-4 text-gray-300 flex-shrink-0" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M8.22 5.22a.75.75 0 0 1 1.06 0l4.25 4.25a.75.75 0 0 1 0 1.06l-4.25 4.25a.75.75 0 0 1-1.06-1.06L11.94 10 8.22 6.28a.75.75 0 0 1 0-1.06Z" clip-rule="evenodd"/>
        </svg>
        @if (!empty($item['url']))
            <a href="{{ $item['url'] }}" class="text-gray-500 hover:text-gray-700 transition-colors truncate max-w-[140px]">
                {{ $item['label'] }}
            </a>
        @else
            <span class="text-gray-900 font-medium truncate max-w-[140px]">{{ $item['label'] }}</span>
        @endif
    @endforeach
</nav>
```

---

## Pagination: `resources/views/components/pagination.blade.php`

```blade
@props(['paginator'])

@if ($paginator->hasPages())
<div class="mt-5 flex flex-col sm:flex-row items-center justify-between gap-3 text-sm">
    <p class="text-gray-500">
        Menampilkan
        <span class="font-medium text-gray-700">{{ $paginator->firstItem() }}</span>
        –
        <span class="font-medium text-gray-700">{{ $paginator->lastItem() }}</span>
        dari
        <span class="font-medium text-gray-700">{{ number_format($paginator->total()) }}</span>
        data
    </p>
    <div>
        {{ $paginator->onEachSide(1)->links('vendor.pagination.tailwind') }}
    </div>
</div>
@endif
```

---

## Form Components

### Input: `resources/views/components/form/input.blade.php`

```blade
@props([
    'name',
    'label'    => null,
    'type'     => 'text',
    'required' => false,
    'hint'     => null,
    'prefix'   => null,
    'suffix'   => null,
])

<div>
    @if ($label)
        <label for="{{ $name }}" class="block text-sm font-medium text-gray-700 mb-1">
            {{ $label }}
            @if ($required) <span class="text-red-500">*</span> @endif
        </label>
    @endif

    <div class="relative">
        @if ($prefix)
            <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                <span class="text-gray-500 text-sm">{{ $prefix }}</span>
            </div>
        @endif

        <input
            id="{{ $name }}"
            name="{{ $name }}"
            type="{{ $type }}"
            {{ $required ? 'required' : '' }}
            {{ $attributes->merge([
                'class' => "block w-full rounded-lg border-gray-300 shadow-sm text-sm
                            focus:ring-2 focus:ring-blue-500 focus:border-blue-500
                            disabled:bg-gray-50 disabled:text-gray-500 disabled:cursor-not-allowed
                            " . ($errors->has($name) ? 'border-red-500 focus:ring-red-500 focus:border-red-500' : '')
                          . ($prefix ? ' pl-8' : '')
                          . ($suffix ? ' pr-8' : ''),
            ]) }}
        >

        @if ($suffix)
            <div class="absolute inset-y-0 right-0 pr-3 flex items-center pointer-events-none">
                <span class="text-gray-500 text-sm">{{ $suffix }}</span>
            </div>
        @endif
    </div>

    @error($name)
        <p class="mt-1 text-xs text-red-600 flex items-center gap-1">
            <svg class="w-3.5 h-3.5 flex-shrink-0" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M18 10a8 8 0 1 1-16 0 8 8 0 0 1 16 0Zm-8-5a.75.75 0 0 1 .75.75v4.5a.75.75 0 0 1-1.5 0v-4.5A.75.75 0 0 1 10 5Zm0 10a1 1 0 1 0 0-2 1 1 0 0 0 0 2Z" clip-rule="evenodd"/>
            </svg>
            {{ $message }}
        </p>
    @enderror

    @if ($hint && !$errors->has($name))
        <p class="mt-1 text-xs text-gray-500">{{ $hint }}</p>
    @endif
</div>
```

**Penggunaan:**
```blade
<x-form.input name="name" label="Nama Produk" :required="true" value="{{ old('name') }}" />
<x-form.input name="price" label="Harga" type="number" prefix="Rp" value="{{ old('price') }}" />
<x-form.input name="email" label="Email" type="email" hint="Gunakan email aktif" />
```

### Select: `resources/views/components/form/select.blade.php`

```blade
@props([
    'name',
    'label'       => null,
    'options'     => [],   // ['value' => 'label'] atau Collection
    'placeholder' => '— Pilih —',
    'required'    => false,
])

<div>
    @if ($label)
        <label for="{{ $name }}" class="block text-sm font-medium text-gray-700 mb-1">
            {{ $label }}
            @if ($required) <span class="text-red-500">*</span> @endif
        </label>
    @endif

    <select
        id="{{ $name }}"
        name="{{ $name }}"
        {{ $required ? 'required' : '' }}
        {{ $attributes->merge([
            'class' => "block w-full rounded-lg border-gray-300 shadow-sm text-sm
                        focus:ring-2 focus:ring-blue-500 focus:border-blue-500
                        disabled:bg-gray-50 disabled:cursor-not-allowed
                        " . ($errors->has($name) ? 'border-red-500' : '')
        ]) }}>

        @if ($placeholder)
            <option value="">{{ $placeholder }}</option>
        @endif

        @foreach ($options as $value => $label)
            <option value="{{ $value }}" {{ old($name) == $value ? 'selected' : '' }}>
                {{ $label }}
            </option>
        @endforeach

        {{ $slot }}
    </select>

    @error($name)
        <p class="mt-1 text-xs text-red-600">{{ $message }}</p>
    @enderror
</div>
```

### Textarea: `resources/views/components/form/textarea.blade.php`

```blade
@props(['name', 'label' => null, 'rows' => 4, 'required' => false, 'hint' => null, 'value' => null])

<div>
    @if ($label)
        <label for="{{ $name }}" class="block text-sm font-medium text-gray-700 mb-1">
            {{ $label }}
            @if ($required) <span class="text-red-500">*</span> @endif
        </label>
    @endif

    <textarea
        id="{{ $name }}"
        name="{{ $name }}"
        rows="{{ $rows }}"
        {{ $required ? 'required' : '' }}
        {{ $attributes->merge([
            'class' => "block w-full rounded-lg border-gray-300 shadow-sm text-sm resize-y
                        focus:ring-2 focus:ring-blue-500 focus:border-blue-500
                        " . ($errors->has($name) ? 'border-red-500' : '')
        ]) }}>{{ old($name, $value) }}</textarea>

    @error($name)
        <p class="mt-1 text-xs text-red-600">{{ $message }}</p>
    @enderror
    @if ($hint && !$errors->has($name))
        <p class="mt-1 text-xs text-gray-500">{{ $hint }}</p>
    @endif
</div>
```

### Checkbox: `resources/views/components/form/checkbox.blade.php`

```blade
@props(['name', 'label', 'hint' => null, 'value' => '1', 'checked' => false])

<div class="flex items-start gap-3">
    <div class="flex items-center h-5">
        <input
            id="{{ $name }}"
            name="{{ $name }}"
            type="hidden"
            value="0">
        <input
            id="{{ $name }}_check"
            name="{{ $name }}"
            type="checkbox"
            value="{{ $value }}"
            {{ old($name, $checked) ? 'checked' : '' }}
            {{ $attributes->merge([
                'class' => 'h-4 w-4 rounded border-gray-300 text-blue-600 focus:ring-blue-500'
            ]) }}>
    </div>
    <div>
        <label for="{{ $name }}_check" class="text-sm font-medium text-gray-700 cursor-pointer">
            {{ $label }}
        </label>
        @if ($hint)
            <p class="text-xs text-gray-500">{{ $hint }}</p>
        @endif
    </div>
</div>
```

### File: `resources/views/components/form/file.blade.php`

```blade
@props([
    'name',
    'label'    => null,
    'accept'   => '*',
    'required' => false,
    'hint'     => null,
    'current'  => null,   // path file yang sudah ada (untuk edit form)
])

<div>
    @if ($label)
        <label for="{{ $name }}" class="block text-sm font-medium text-gray-700 mb-1">
            {{ $label }}
            @if ($required) <span class="text-red-500">*</span> @endif
        </label>
    @endif

    {{-- Preview file yang sudah ada (edit form) --}}
    @if ($current)
        <div class="mb-2 flex items-center gap-3">
            @if (str_starts_with(mime_content_type(storage_path('app/public/' . $current)) ?? '', 'image/'))
                <img src="{{ Storage::url($current) }}"
                     alt="Current file"
                     class="w-16 h-16 rounded-lg object-cover border border-gray-200">
            @else
                <div class="flex items-center gap-2 text-sm text-gray-600">
                    <svg class="w-5 h-5 text-gray-400" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m2.25 0H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z"/>
                    </svg>
                    {{ basename($current) }}
                </div>
            @endif
            <span class="text-xs text-gray-400">File saat ini</span>
        </div>
    @endif

    <input
        id="{{ $name }}"
        name="{{ $name }}"
        type="file"
        accept="{{ $accept }}"
        {{ $required && !$current ? 'required' : '' }}
        {{ $attributes->merge([
            'class' => "block w-full text-sm text-gray-500
                        file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0
                        file:text-sm file:font-medium file:bg-blue-50 file:text-blue-700
                        hover:file:bg-blue-100 cursor-pointer
                        " . ($errors->has($name) ? 'border border-red-500 rounded-lg p-1' : '')
        ]) }}
    >

    @error($name)
        <p class="mt-1 text-xs text-red-600 flex items-center gap-1">
            <svg class="w-3.5 h-3.5 flex-shrink-0" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M18 10a8 8 0 1 1-16 0 8 8 0 0 1 16 0Zm-8-5a.75.75 0 0 1 .75.75v4.5a.75.75 0 0 1-1.5 0v-4.5A.75.75 0 0 1 10 5Zm0 10a1 1 0 1 0 0-2 1 1 0 0 0 0 2Z" clip-rule="evenodd"/>
            </svg>
            {{ $message }}
        </p>
    @enderror

    @if ($hint && !$errors->has($name))
        <p class="mt-1 text-xs text-gray-500">{{ $hint }}</p>
    @endif
</div>
```

**Penggunaan:**
```blade
{{-- Upload gambar baru --}}
<x-form.file name="photo" label="Foto Produk" accept="image/*" :required="true"
    hint="Format: JPG, PNG, WebP. Maks 2MB." />

{{-- Edit: tampilkan file saat ini + opsi ganti --}}
<x-form.file name="photo" label="Foto Produk" accept="image/*"
    :current="$entity?->photo_path"
    hint="Kosongkan jika tidak ingin mengganti foto." />
```

> **Catatan:** Form yang menggunakan file upload WAJIB menambahkan `enctype="multipart/form-data"` di tag `<form>`.

---

## Dashboard: `resources/views/dashboard/index.blade.php`

Contoh dashboard lengkap dengan stat cards.

```blade
<x-layouts.app title="Dashboard"
    :breadcrumbs="[['label' => 'Dashboard']]">

    <x-slot name="header">
        <div class="flex items-center justify-between">
            <div>
                <h1 class="text-xl font-bold text-gray-900">Dashboard</h1>
                <p class="text-sm text-gray-500 mt-0.5">Selamat datang, {{ auth()->user()->name }}</p>
            </div>
            <span class="text-sm text-gray-400">{{ now()->translatedFormat('l, d F Y') }}</span>
        </div>
    </x-slot>

    {{-- Stat Cards --}}
    <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-6 mb-8">
        <x-stat-card label="Total Produk" value="0" change="Belum ada data" trend="neutral" color="blue">
            <x-slot name="icon">
                <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M21 7.5l-9-5.25L3 7.5m18 0l-9 5.25m9-5.25v9l-9 5.25M3 7.5l9 5.25M3 7.5v9l9 5.25m0-9v9"/>
                </svg>
            </x-slot>
        </x-stat-card>

        <x-stat-card label="Total Pesanan" value="0" change="Belum ada data" trend="neutral" color="emerald">
            <x-slot name="icon">
                <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M2.25 3h1.386c.51 0 .955.343 1.087.835l.383 1.437M7.5 14.25a3 3 0 0 0-3 3h15.75m-12.75-3h11.218c1.121-2.3 2.1-4.684 2.924-7.138a60.114 60.114 0 0 0-16.536-1.84M7.5 14.25 5.106 5.272M6 20.25a.75.75 0 1 1-1.5 0 .75.75 0 0 1 1.5 0Zm12.75 0a.75.75 0 1 1-1.5 0 .75.75 0 0 1 1.5 0Z"/>
                </svg>
            </x-slot>
        </x-stat-card>

        <x-stat-card label="Total Pelanggan" value="0" change="Belum ada data" trend="neutral" color="violet">
            <x-slot name="icon">
                <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M15 19.128a9.38 9.38 0 0 0 2.625.372 9.337 9.337 0 0 0 4.121-.952 4.125 4.125 0 0 0-7.533-2.493M15 19.128v-.003c0-1.113-.285-2.16-.786-3.07M15 19.128v.106A12.318 12.318 0 0 1 8.624 21c-2.331 0-4.512-.645-6.374-1.766l-.001-.109a6.375 6.375 0 0 1 11.964-3.07M12 6.375a3.375 3.375 0 1 1-6.75 0 3.375 3.375 0 0 1 6.75 0Zm8.25 2.25a2.625 2.625 0 1 1-5.25 0 2.625 2.625 0 0 1 5.25 0Z"/>
                </svg>
            </x-slot>
        </x-stat-card>

        <x-stat-card label="Pendapatan" value="Rp 0" change="Belum ada data" trend="neutral" color="amber">
            <x-slot name="icon">
                <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M12 6v12m-3-2.818.879.659c1.171.879 3.07.879 4.242 0 1.172-.879 1.172-2.303 0-3.182C13.536 12.219 12.768 12 12 12c-.725 0-1.45-.22-2.003-.659-1.106-.879-1.106-2.303 0-3.182s2.9-.879 4.006 0l.415.33M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z"/>
                </svg>
            </x-slot>
        </x-stat-card>
    </div>

    {{-- Quick Links --}}
    <x-card title="Menu Cepat">
        <p class="text-sm text-gray-500">
            Modul belum tersedia. Jalankan <code class="bg-gray-100 px-1.5 py-0.5 rounded text-xs font-mono">/laravel-blade new-module</code> untuk menambah modul.
        </p>
    </x-card>

</x-layouts.app>
```

---

## Contoh Halaman Index Lengkap

Lihat bagian table di `templates/module-slice.md`. Dengan komponen di atas, tampilan akhir:

```
┌────────────────────────────────────────────────────────────────────┐
│ [Produk]  ▸  [Daftar Produk]           (breadcrumb di navbar)     │
├────────────────────────────────────────────────────────────────────┤
│  Daftar Produk                                         [+ Tambah] │
│  ────────────────────────────────────────────────────────────────  │
│  🔍 [  Cari produk...  ] [Cari]  [Reset]                          │
│  ┌──┬────────────────┬───────────────┬─────────┬─────────────────┐│
│  │# │ NAMA           │ STATUS        │ HARGA   │ AKSI            ││
│  ├──┼────────────────┼───────────────┼─────────┼─────────────────┤│
│  │1 │ Laptop Gaming  │ ● Aktif       │ Rp 15M  │ Detail Edit ✕   ││
│  │2 │ Mouse Wireless │ ○ Nonaktif    │ Rp 250k │ Detail Edit ✕   ││
│  └──┴────────────────┴───────────────┴─────────┴─────────────────┘│
│                                                                    │
│  Menampilkan 1–15 dari 84 data      [← 1  2  3  4  5 →]          │
└────────────────────────────────────────────────────────────────────┘
```
