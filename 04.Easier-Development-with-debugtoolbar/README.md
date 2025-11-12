# 04: Easier Development with debugtoolbar

## Deskripsi

Proyek ini mendemonstrasikan penggunaan `pyramid_debugtoolbar` untuk memudahkan proses development dan debugging aplikasi Pyramid. Debugtoolbar adalah add-on Pyramid yang menyediakan berbagai tools debugging langsung di browser, termasuk traceback yang informatif ketika terjadi error.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini juga memperkenalkan konsep **Setuptools extras** untuk mengelola dependencies development secara terpisah dari dependencies production.

## Background

### Error Handling dan Introspection

Selama development, kita membutuhkan tools yang membantu:
- **Error handling**: Menampilkan traceback yang jelas ketika terjadi error
- **Introspection**: Memeriksa variabel, request, response, dan informasi debugging lainnya
- **Development tools**: Template reloading, application reloading, dan tools lainnya

### Pyramid Add-ons

`pyramid_debugtoolbar` adalah contoh dari **Pyramid add-on**, yaitu package Python yang menambahkan fungsionalitas ke aplikasi Pyramid. Add-on dapat dikonfigurasi dengan dua cara:
1. **Imperative configuration**: Menggunakan `config.include()` di kode Python
2. **Declarative configuration**: Menggunakan `pyramid.includes` di file `.ini`

### Setuptools Extras

Setuptools extras memungkinkan kita mendefinisikan dependencies yang **opsional** atau **khusus untuk environment tertentu** (seperti development, testing, atau production). Ini memungkinkan:
- Dependencies production tetap ringan
- Dependencies development terpisah dan tidak terinstall di production
- Fleksibilitas dalam mengelola dependencies

## Tujuan Pembelajaran

1. Memahami cara menginstall dan mengaktifkan `pyramid_debugtoolbar`
2. Memahami konsep Pyramid add-ons
3. Memahami cara mengkonfigurasi add-on melalui file `.ini`
4. Memahami konsep Setuptools extras untuk development dependencies
5. Menggunakan debugtoolbar untuk debugging dan introspection

## Struktur Proyek

```
04.Easier-Development-with-debugtoolbar/
├── setup.py                 # Setup file dengan extras_require untuk dev dependencies
├── development.ini          # File konfigurasi dengan pyramid.includes
├── requirements.txt         # Dependencies production
├── tutorial/                # Package aplikasi
│   └── __init__.py         # Berisi fungsi main() sebagai WSGI factory
└── tutorial.egg-info/      # Metadata package (auto-generated)
```

## Persyaratan

- Python 3.6 atau lebih baru
- pip (Python package manager)
- Virtual environment (disarankan)

## Instalasi

1. **Buat virtual environment (jika belum ada):**

   ```bash
   python -m venv venv
   ```

2. **Aktifkan virtual environment:**

   **Windows:**
   ```bash
   venv\Scripts\activate
   ```

   **Linux/Mac:**
   ```bash
   source venv/bin/activate
   ```

3. **Install dependencies production:**

   ```bash
   pip install -r requirements.txt
   ```

4. **Install package dengan development dependencies:**

   ```bash
   pip install -e ".[dev]"
   ```

   **Penjelasan perintah:**
   - `-e`: Install dalam mode editable (perubahan kode langsung terlihat)
   - `.[dev]`: Install package dengan extras `dev`
   - Tanda kutip diperlukan karena `[dev]` memiliki karakter khusus di shell

   Perintah ini akan:
   - Install dependencies production (`pyramid`, `waitress`)
   - Install dependencies development (`pyramid_debugtoolbar`)
   - Install package `tutorial` dalam mode editable
   - Generate folder `tutorial.egg-info/` yang berisi metadata package

## Menjalankan Aplikasi

1. **Jalankan dengan pserve:**

   ```bash
   pserve development.ini --reload
   ```

   Flag `--reload` akan:
   - Watch filesystem untuk perubahan pada file Python dan `.ini`
   - Otomatis restart aplikasi ketika ada perubahan
   - Sangat berguna untuk development

2. **Buka browser:**

   Navigate ke: http://localhost:6543/

   Anda akan melihat:
   - Halaman dengan teks "Hello World!"
   - **Toolbar debug di sisi kanan browser** (fitur baru dari debugtoolbar)

3. **Coba fitur debugtoolbar:**

   - Klik tombol toolbar di sisi kanan untuk melihat informasi debugging
   - Coba akses variabel, request, response, dan informasi lainnya
   - Lihat performance metrics dan SQL queries (jika ada)

4. **Stop aplikasi:**

   Tekan `Ctrl+C` di terminal

## Penjelasan File

### setup.py

File ini mendefinisikan package Python dengan **Setuptools extras**:

```python
requires = [
    'pyramid',
    'waitress',
]

dev_requires = [
    'pyramid_debugtoolbar',
]

setup(
    name='tutorial',
    install_requires=requires,
    extras_require={
        'dev': dev_requires,
    },
    entry_points={
        'paste.app_factory': [
            'main = tutorial:main'
        ],
    },
)
```

**Penjelasan:**

- **`requires`**: Dependencies yang selalu diinstall (production dependencies)
- **`dev_requires`**: Dependencies khusus untuk development
- **`extras_require`**: Dictionary yang mendefinisikan extras (opsional dependencies)
  - Key `'dev'`: Nama extra
  - Value `dev_requires`: List dependencies untuk extra tersebut

**Cara menggunakan extras:**

- Install tanpa extras: `pip install -e .` (hanya install `requires`)
- Install dengan extras dev: `pip install -e ".[dev]"` (install `requires` + `dev_requires`)

**Mengapa menggunakan extras?**

- Production environment tidak perlu install development tools
- Development dependencies terpisah dan jelas
- Fleksibilitas dalam mengelola dependencies

### development.ini

File konfigurasi yang mengaktifkan debugtoolbar melalui `pyramid.includes`:

```ini
[app:main]
use = egg:tutorial
pyramid.includes =
    pyramid_debugtoolbar

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

**Penjelasan:**

- **`pyramid.includes`**: Directive Pyramid untuk mengaktifkan add-on
  - Format: Multi-line list (setiap baris adalah nama add-on)
  - `pyramid_debugtoolbar`: Nama add-on yang akan diaktifkan

**Cara kerja:**

1. Pyramid membaca `pyramid.includes` dari `.ini` file
2. Pyramid mencari dan memanggil konfigurasi dari add-on tersebut
3. Add-on menambahkan fungsionalitas ke aplikasi (dalam hal ini, debugtoolbar)

**Alternatif: Imperative Configuration**

Anda juga bisa mengaktifkan add-on di kode Python:

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_debugtoolbar')  # Aktifkan di sini
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

**Keuntungan menggunakan `.ini` file:**

- Tidak perlu mengubah kode untuk enable/disable toolbar
- Mudah diaktifkan/dinonaktifkan untuk environment berbeda
- Separation of concerns: konfigurasi terpisah dari kode

### tutorial/__init__.py

Berisi fungsi `main()` yang merupakan WSGI application factory:

```python
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):
    return Response('<body><h1>Hello World!</h1></body>')


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

**Tidak ada perubahan dari tutorial sebelumnya** - debugtoolbar dikonfigurasi melalui `.ini` file, bukan di kode.

## Fitur Debugtoolbar

Setelah debugtoolbar diaktifkan, Anda akan melihat:

1. **Toolbar di Browser:**
   - Tombol di sisi kanan browser
   - Klik untuk membuka panel debugging

2. **Panel Debugging:**
   - **Request/Response**: Informasi tentang HTTP request dan response
   - **Variables**: Variabel dalam scope saat ini
   - **Templates**: Template yang digunakan
   - **Performance**: Metrics performa aplikasi
   - **SQL**: Query database (jika menggunakan database)
   - **History**: History request sebelumnya

3. **Error Traceback:**
   - Ketika terjadi error, debugtoolbar menampilkan traceback yang informatif
   - Dapat melihat nilai variabel di setiap level stack
   - Dapat mengevaluasi ekspresi Python dalam konteks error

## Menonaktifkan Debugtoolbar

Untuk menonaktifkan debugtoolbar, cukup hapus atau comment baris di `development.ini`:

```ini
[app:main]
use = egg:tutorial
# pyramid.includes =
#     pyramid_debugtoolbar
```

Atau hapus sepenuhnya:

```ini
[app:main]
use = egg:tutorial
```

**Tidak perlu mengubah kode Python!** Ini menunjukkan keuntungan menggunakan konfigurasi `.ini`.

## Catatan Penting

### HTML/CSS Injection

Debugtoolbar menyuntikkan sejumlah kecil HTML/CSS ke aplikasi (tepat sebelum tag `</body>`) untuk menampilkan toolbar. Jika Anda mengalami masalah client-side yang tidak dapat dijelaskan, coba nonaktifkan debugtoolbar sementara.

### Production Environment

**JANGAN gunakan debugtoolbar di production!** Debugtoolbar:
- Menampilkan informasi sensitif (variabel, stack trace, dll)
- Menambahkan overhead performa
- Hanya untuk development

Pastikan file konfigurasi production (misalnya `production.ini`) tidak memiliki `pyramid_debugtoolbar` di `pyramid.includes`.

## Troubleshooting

### Error: "No module named 'pyramid_debugtoolbar'"

**Solusi:** Pastikan Anda sudah menjalankan `pip install -e ".[dev]"` (dengan extras `[dev]`)

### Toolbar tidak muncul di browser

**Solusi:**
1. Pastikan `pyramid.includes` di `development.ini` sudah benar
2. Pastikan `pyramid_debugtoolbar` sudah terinstall
3. Restart aplikasi dengan `pserve development.ini --reload`
4. Clear cache browser

### Error: "Entry point 'main' not found"

**Solusi:** Pastikan `setup.py` memiliki entry point yang benar dan sudah di-install ulang dengan `pip install -e ".[dev]"`

### Port sudah digunakan

**Solusi:** Ubah port di `development.ini` atau stop aplikasi yang menggunakan port tersebut

### Perubahan kode tidak terlihat

**Solusi:** Pastikan menggunakan flag `--reload` atau restart aplikasi secara manual

## Extra Credit

### 1. Mengapa pyramid_debugtoolbar ditambahkan ke dev_requires, bukan requires?

**Jawaban:**

`pyramid_debugtoolbar` adalah tool development yang:
- Hanya dibutuhkan saat development, tidak di production
- Menambahkan overhead performa
- Menampilkan informasi sensitif yang tidak boleh ada di production

Dengan menempatkannya di `dev_requires`, kita memastikan:
- Production environment tidak menginstall tool yang tidak diperlukan
- Dependencies production tetap ringan
- Clear separation antara production dan development dependencies

### 2. Coba buat bug di aplikasi

Ubah fungsi `hello_world` menjadi:

```python
def hello_world(request):
    return xResponse('<body><h1>Hello World!</h1></body>')  # xResponse tidak ada
```

Simpan dan refresh browser. Perhatikan:
- Traceback yang informatif ditampilkan oleh debugtoolbar
- Klik icon "screen" di baris terakhir traceback
- Coba ketik variabel `request` dan `Response` di interactive console
- Jelajahi fitur-fitur debugging lainnya

**Apa yang bisa Anda temukan?**
- Nilai variabel di setiap level stack
- Request object dengan semua atributnya
- Response object
- Template context
- Performance metrics
- Dan banyak lagi!

### 3. Apa perbedaan antara config.include() dan pyramid.includes?

**Jawaban:**

- **`config.include()`**: Imperative configuration di kode Python
  - Harus mengubah kode untuk enable/disable
  - Lebih eksplisit dan terlihat di kode
  - Cocok untuk add-on yang selalu diperlukan

- **`pyramid.includes`**: Declarative configuration di file `.ini`
  - Tidak perlu mengubah kode
  - Mudah diaktifkan/dinonaktifkan untuk environment berbeda
  - Cocok untuk add-on opsional seperti debugtoolbar

Kedua cara menghasilkan hasil yang sama, tapi `.ini` file memberikan fleksibilitas lebih.

## Referensi

- [Pyramid Documentation - Debugtoolbar](https://docs.pylonsproject.org/projects/pyramid-debugtoolbar/en/latest/)
- [Pyramid Documentation - Configuration](https://docs.pylonsproject.org/projects/pyramid/en/latest/)
- [Setuptools Extras](https://setuptools.readthedocs.io/en/latest/userguide/declarative_config.html#options)
- [Pyramid Add-ons](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/extending.html)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.

