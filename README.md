# MVC-Code-Igniter

# Laporan Praktikum: Instalasi & Pengembangan Web Dasar dengan CodeIgniter 4

---

## 1. Persiapan Lingkungan Pengembangan (XAMPP & PHP)

### 1.1 Instalasi XAMPP
Untuk menjalankan CodeIgniter 4 secara lokal, diperlukan *runtime* PHP yang didapatkan melalui paket bundling **XAMPP** (PHP 8.2.12). 
* **Kendala:** Pengunduhan melalui browser maupun `curl` dari server SourceForge mengalami pembatasan *bandwidth* (~60–80 KB/detik).
* **Solusi:** Instalasi dilakukan melalui terminal menggunakan *Windows Package Manager* (`winget`) agar mendapatkan jalur CDN resmi yang lebih cepat:
  ```powershell
  winget install ApacheFriends.Xampp.8.2
  ```

![Instalasi XAMPP via Winget]("<permalink gambar yang nanti saya input>")

### 1.2 Konfigurasi Environment Variables (Path) & Ekstensi PHP
1. Menambahkan direktori `C:\xampp\php` ke dalam **System Environment Variables (`Path`)** agar perintah `php` dapat diakses secara global melalui terminal.
2. Melakukan *restart* terminal dan memverifikasi instalasi dengan perintah:
   ```powershell
   php -v
   ```
3. Mengaktifkan ekstensi wajib CodeIgniter 4 pada file `C:\xampp\php\php.ini` dengan menghapus tanda titik koma (`;`):
   ```ini
   extension=intl
   extension=mbstring
   extension=curl
   ```

![Verifikasi Versi PHP di Terminal]("<permalink gambar yang nanti saya input>")

---

## 2. Konfigurasi Awal Proyek CodeIgniter 4

Proyek menggunakan **CodeIgniter v4.7.6** yang diunduh langsung dari situs resmi dan diekstrak ke direktori lokal.

1. Menyalin template konfigurasi `env` menjadi `.env`:
   ```cmd
   copy env .env
   ```
2. Mengubah mode lingkungan menjadi `development` agar pesan *error* dan *debugging* ditampilkan secara rinci saat proses pengembangan, serta mengatur `app.baseURL`:
   ```ini
   CI_ENVIRONMENT = development
   app.baseURL = 'http://localhost:8080/'
   ```
3. Menjalankan server pengembangan bawaan melalui CLI Spark:
   ```bash
   php spark serve
   ```

![Tampilan Awal CodeIgniter 4 di Localhost]("<permalink gambar yang nanti saya input>")

---

## 3. Implementasi Controller dan Routing

### 3.1 Pembuatan Controller `Page.php`
Membuat controller baru pada `app/Controllers/Page.php` untuk menangani beberapa halaman statis (`about`, `contact`, `faqs`, `tos`).

```php
<?php namespace App\Controllers;

class Page extends BaseController
{
    public function about()
    {
        echo "about page";
    }

    public function contact()
    {
        echo "contact page";
    }

    public function faqs()
    {
        echo "Faqs page";
    }

    public function tos()
    {
        echo "Halaman Term of Service";
    }
}
```

### 3.2 Penanganan Kendala Routing (Auto Route vs Defined Routes)
* **Kendala:** Pada CodeIgniter v4.7.6, fitur *Auto Routing (Legacy)* telah dihapus demi keamanan. Sistem menerapkan *Auto Routing (Improved)* yang mewajibkan awalan metode HTTP (seperti `getTos()`) serta melarang akses *auto route* pada controller yang sudah didaftarkan di *Defined Routes* (`Cannot access the controller in Defined Routes`).
* **Solusi:** Mendaftarkan seluruh rute halaman secara eksplisit di dalam file `app/Config/Routes.php`:

```php
<?php

use CodeIgniter\Router\RouteCollection;

/** @var RouteCollection $routes */
$routes->get('/', 'Home::index');
$routes->get('/about', 'Page::about');
$routes->get('/contact', 'Page::contact');
$routes->get('/faqs', 'Page::faqs');
$routes->get('/page/tos', 'Page::tos');
```

![Konfigurasi Routes.php dan Hasil Uji Halaman]("<permalink gambar yang nanti saya input>")

---

## 4. Integrasi Bootstrap 5 dan Modular View

Untuk membangun antarmuka yang responsif tanpa mengulang kode HTML pada setiap halaman, struktur tampilan dipisah menjadi komponen `header`, konten utama, dan `footer` di dalam direktori `app/Views/`.

### 4.1 Pembaruan Controller `Page.php`
Mengubah *output* masing-masing *method* pada `app/Controllers/Page.php` dari `echo` menjadi pemanggilan fungsi `view()`:

```php
<?php

namespace App\Controllers;

class Page extends BaseController
{
    public function index()
    {
        return view('welcome_message');
    }

    public function about()
    {
        return view('about');
    }

    public function contact()
    {
        return view('contact');
    }

    public function faqs()
    {
        return view('faqs');
    }
}
```

### 4.2 Pembuatan Template Layout (`header.php` & `footer.php`)
Membuat folder `app/Views/layout/` berisi dua file template utama menggunakan **Bootstrap 5.3.3 CDN**:

**`app/Views/layout/header.php`**
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Website CodeIgniter 4</title>
    <link href="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css](https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css)" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="<?= base_url() ?>">MyWeb</a>
            <div class="navbar-nav">
                <a class="nav-link" href="<?= base_url('about') ?>">About</a>
                <a class="nav-link" href="<?= base_url('contact') ?>">Contact</a>
                <a class="nav-link" href="<?= base_url('faqs') ?>">FAQs</a>
            </div>
        </div>
    </nav>
    <div class="container mt-4">
```

**`app/Views/layout/footer.php`**
```html
    </div>
    <footer class="text-center py-4 text-muted mt-5 border-top">
        <p>&copy; <?= date('Y') ?> Tutorial CI4 Bootstrap</p>
    </footer>

    <script src="[https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js](https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js)"></script>
</body>
</html>
```

### 4.3 Pembuatan File View Halaman & Penyelesaian `ViewException`
* **Kendala:** Muncul pesan error `CodeIgniter\View\Exceptions\ViewException: Invalid file: "about.php"` dan `"contact.php"` karena file view belum diletakkan langsung di dalam root direktori `app/Views/`.
* **Solusi:** Memastikan file `about.php` dan `contact.php` berada tepat di jalur `app/Views/` (sejajar dengan folder `layout`) dengan ekstensi `.php` yang valid.

**`app/Views/about.php`**
```php
<?= view('layout/header') ?>

<div class="p-5 mb-4 bg-light rounded-3">
    <div class="container-fluid py-5">
        <h1 class="display-5 fw-bold">Tentang Kami</h1>
        <p class="col-md-8 fs-4">Ini adalah halaman about yang didesain menggunakan komponen Bootstrap di CodeIgniter 4.</p>
        <button class="btn btn-primary btn-lg" type="button">Pelajari Lebih Lanjut</button>
    </div>
</div>

<?= view('layout/footer') ?>
```

**`app/Views/contact.php`**
```php
<?= view('layout/header') ?>

<h2>Hubungi Kami</h2>
<form class="mt-3">
    <div class="mb-3">
        <label class="form-label">Nama</label>
        <input type="text" class="form-control" placeholder="Masukkan nama Anda">
    </div>
    <div class="mb-3">
        <label class="form-label">Pesan</label>
        <textarea class="form-control" rows="4" placeholder="Tuliskan pesan Anda"></textarea>
    </div>
    <button type="submit" class="btn btn-success">Kirim Pesan</button>
</form>

<?= view('layout/footer') ?>
```

![Hasil Tampilan Halaman About dengan Bootstrap]("<permalink gambar yang nanti saya input>")

![Hasil Tampilan Halaman Contact dengan Bootstrap]("<permalink gambar yang nanti saya input>")

---

## 5. Kesimpulan
Seluruh proses konfigurasi lingkungan lokal (PHP 8.2.12 via XAMPP), pengaturan *routing* eksplisit pada CodeIgniter v4.7.6, serta pemisahan tata letak antarmuka (*modular view*) menggunakan Bootstrap 5 berhasil diimplementasikan dan dijalankan melalui server lokal `http://localhost:8080`.
