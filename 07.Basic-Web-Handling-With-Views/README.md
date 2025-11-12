# 07: Basic Web Handling With Views

## Deskripsi

Proyek ini mendemonstrasikan cara mengorganisir views ke dalam modul terpisah menggunakan decorators untuk konfigurasi deklaratif di Pyramid. Tutorial ini memperkenalkan konsep memisahkan views dari kode startup aplikasi dan menggunakan `@view_config` decorator untuk mendeklarasikan konfigurasi views.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Memindahkan views ke modul terpisah
- Menggunakan `config.scan()` untuk menemukan views secara otomatis
- Menggunakan `@view_config` decorator untuk konfigurasi deklaratif
- Membuat multiple views dengan routing yang berbeda
- Testing multiple views dengan unit tests dan functional tests

## Background

### Views dalam Pyramid

Dalam Pyramid, **views** adalah cara utama untuk menerima web requests dan mengembalikan responses. Sebuah view adalah callable yang menerima request sebagai parameter dan mengembalikan response.

### Imperative vs Declarative Configuration

Pyramid mendukung dua pendekatan untuk konfigurasi views:

**Imperative Configuration:**
- Menggunakan `config.add_view()` secara eksplisit
- Konfigurasi terpisah dari definisi view function
- Contoh:
  ```python
  config.add_view(hello_world, route_name='hello')
  ```

**Declarative Configuration:**
- Menggunakan `@view_config` decorator
- Konfigurasi terletak langsung di atas view function
- Lebih deklaratif dan mudah dibaca
- Contoh:
  ```python
  @view_config(route_name='hello')
  def hello_world(request):
      return Response('Hello World')
  ```

Kedua pendekatan menghasilkan konfigurasi yang sama. Pilihan biasanya berdasarkan preferensi dan kebiasaan tim.

### Config.scan()

`config.scan()` adalah metode yang memungkinkan Pyramid untuk secara otomatis menemukan dan mendaftarkan views yang menggunakan decorators. Pyramid akan:
- Memindai modul yang ditentukan
- Mencari functions/classes dengan decorators `@view_config`
- Secara otomatis mendaftarkan views tersebut

## Tujuan Pembelajaran

1. Memahami cara memisahkan views dari kode startup aplikasi
2. Menggunakan `config.scan()` untuk menemukan views secara otomatis
3. Menggunakan `@view_config` decorator untuk konfigurasi deklaratif
4. Membuat multiple views dengan routing yang berbeda
5. Menguji multiple views dengan unit tests dan functional tests

## Struktur Proyek

```
07.Basic-Web-Handling-With-Views/
├── setup.py                 # Setup file dengan dependencies
├── development.ini          # File konfigurasi aplikasi
├── requirements.txt         # Dependencies production
├── tutorial/                # Package aplikasi
│   ├── __init__.py          # Berisi fungsi main() dengan scan
│   ├── views.py             # Modul views dengan decorators
│   └── tests.py             # File unit tests dan functional tests
└── tutorial.egg-info/       # Metadata package (auto-generated)
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

3. **Install package dengan development dependencies:**

   ```bash
   pip install -e ".[dev]"
   ```

   **Penjelasan perintah:**
   - `-e`: Install dalam mode editable (perubahan kode langsung terlihat)
   - `.[dev]`: Install package dengan extras `dev`
   - Tanda kutip diperlukan karena `[dev]` memiliki karakter khusus di shell

   Perintah ini akan:
   - Install dependencies production (`pyramid`, `waitress`)
   - Install dependencies development (`pyramid_debugtoolbar`, `pytest`, `webtest`)
   - Install package `tutorial` dalam mode editable

## Penggunaan

### Menjalankan Aplikasi

Setelah instalasi selesai, jalankan aplikasi dengan perintah:

```bash
pserve development.ini --reload
```

Aplikasi akan berjalan di `http://localhost:6543/`

### Mengakses Views

- **Home View:** `http://localhost:6543/`
  - Menampilkan halaman home dengan link ke hello view
  - Route name: `home`

- **Hello View:** `http://localhost:6543/howdy`
  - Menampilkan halaman hello dengan link kembali ke home
  - Route name: `hello`

## Struktur Kode

### tutorial/__init__.py

File ini berisi fungsi `main()` yang mengkonfigurasi aplikasi Pyramid:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan:**
- `config.add_route('home', '/')`: Menambahkan route `home` yang map ke URL `/`
- `config.add_route('hello', '/howdy')`: Menambahkan route `hello` yang map ke URL `/howdy`
- `config.scan('.views')`: Memindai modul `views` untuk menemukan views dengan decorators
- Titik (`.`) dalam `.views` berarti modul relatif terhadap package saat ini

### tutorial/views.py

File ini berisi views yang menggunakan `@view_config` decorator:

```python
from pyramid.response import Response
from pyramid.view import view_config


# First view, available at http://localhost:6543/
@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')


# /howdy
@view_config(route_name='hello')
def hello(request):
    return Response('<body>Go back <a href="/">home</a></body>')
```

**Penjelasan:**
- `@view_config(route_name='home')`: Decorator yang mendeklarasikan view ini terhubung ke route `home`
- `@view_config(route_name='hello')`: Decorator yang mendeklarasikan view ini terhubung ke route `hello`
- Setiap view function menerima `request` sebagai parameter
- Setiap view mengembalikan `Response` object

### tutorial/tests.py

File ini berisi unit tests dan functional tests untuk kedua views:

```python
import unittest

from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_home(self):
        from .views import home

        request = testing.DummyRequest()
        response = home(request)
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Visit', response.body)

    def test_hello(self):
        from .views import hello

        request = testing.DummyRequest()
        response = hello(request)
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'Go back', response.body)


class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp

        self.testapp = TestApp(app)

    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<body>Visit', res.body)

    def test_hello(self):
        res = self.testapp.get('/howdy', status=200)
        self.assertIn(b'<body>Go back', res.body)
```

**Penjelasan:**
- `TutorialViewTests`: Unit tests yang menguji views secara terisolasi
- `TutorialFunctionalTests`: Functional tests yang menguji views melalui HTTP requests
- `test_home()`: Menguji home view
- `test_hello()`: Menguji hello view
- `assertIn()`: Menguji apakah string tertentu ada dalam response body

## Menjalankan Tests

Untuk menjalankan tests, gunakan perintah:

```bash
pytest tutorial/tests.py -q
```

Atau dengan verbose output:

```bash
pytest tutorial/tests.py -v
```

**Output yang diharapkan:**
```
.... 4 passed in 0.28 seconds
```

## Konsep Penting

### 1. Config.scan()

`config.scan()` memungkinkan Pyramid untuk secara otomatis menemukan dan mendaftarkan views:

```python
config.scan('.views')
```

- Titik (`.`) berarti modul relatif terhadap package saat ini
- Pyramid akan memindai modul `views` untuk mencari decorators
- Views dengan `@view_config` akan otomatis didaftarkan

### 2. @view_config Decorator

Decorator ini mendeklarasikan konfigurasi view:

```python
@view_config(route_name='home')
def home(request):
    return Response('...')
```

- `route_name`: Nama route yang terhubung ke view ini
- Decorator ini membuat konfigurasi deklaratif
- Lebih mudah dibaca dan dipelihara

### 3. Multiple Views

Kita dapat membuat multiple views dengan routing yang berbeda:

- Setiap view memiliki route name yang unik
- Setiap view dapat memiliki URL pattern yang berbeda
- Nama view function, route name, dan URL dapat berbeda

### 4. Testing Multiple Views

Kita dapat menguji multiple views dengan:

- **Unit Tests:** Menguji views secara terisolasi
- **Functional Tests:** Menguji views melalui HTTP requests
- Setiap view harus diuji secara terpisah

## Analisis

### Perbedaan dari Tutorial Sebelumnya

1. **Views Dipisahkan:** Views sekarang berada di modul terpisah (`views.py`)
2. **Konfigurasi Deklaratif:** Menggunakan `@view_config` decorator
3. **Config.scan():** Menggunakan `config.scan()` untuk menemukan views
4. **Multiple Views:** Memiliki dua views (home dan hello)
5. **Routing Berbeda:** Setiap view memiliki route dan URL yang berbeda

### Keuntungan Pendekatan Ini

1. **Organisasi Kode:** Views terpisah dari kode startup
2. **Mudah Dibaca:** Konfigurasi deklaratif lebih mudah dipahami
3. **Skalabilitas:** Mudah menambahkan views baru
4. **Maintainability:** Lebih mudah memelihara kode

### Nama Route vs URL vs View Function

- **Route Name:** `home`, `hello` (nama internal untuk routing)
- **URL:** `/`, `/howdy` (URL yang diakses user)
- **View Function:** `home()`, `hello()` (nama function)

Ketiga hal ini dapat berbeda, memberikan fleksibilitas dalam organisasi kode.

## Analisis

### Perbedaan dari Tutorial Sebelumnya

1. **Views Dipisahkan:** Views sekarang berada di modul terpisah (`views.py`)
2. **Konfigurasi Deklaratif:** Menggunakan `@view_config` decorator
3. **Config.scan():** Menggunakan `config.scan()` untuk menemukan views
4. **Multiple Views:** Memiliki dua views (home dan hello)
5. **Routing Berbeda:** Setiap view memiliki route dan URL yang berbeda

### Keuntungan Pendekatan Ini

1. **Organisasi Kode:** Views terpisah dari kode startup
2. **Mudah Dibaca:** Konfigurasi deklaratif lebih mudah dipahami
3. **Skalabilitas:** Mudah menambahkan views baru
4. **Maintainability:** Lebih mudah memelihara kode

### Nama Route vs URL vs View Function

- **Route Name:** `home`, `hello` (nama internal untuk routing)
- **URL:** `/`, `/howdy` (URL yang diakses user)
- **View Function:** `home()`, `hello()` (nama function)

Ketiga hal ini dapat berbeda, memberikan fleksibilitas dalam organisasi kode.

### Kesimpulan

Tutorial ini memperkenalkan konsep penting dalam Pyramid:

1. **Memisahkan Views:** Views dapat dipisahkan ke modul terpisah
2. **Konfigurasi Deklaratif:** Menggunakan `@view_config` decorator
3. **Config.scan():** Menggunakan `config.scan()` untuk menemukan views
4. **Multiple Views:** Membuat multiple views dengan routing yang berbeda
5. **Testing:** Menguji multiple views dengan unit tests dan functional tests

Konsep-konsep ini merupakan dasar untuk membangun aplikasi Pyramid yang lebih kompleks dan terorganisir dengan baik.

## Extra Credit

### 1. Apa arti titik (`.`) dalam `.views`?

**Jawaban:**

Titik (`.`) dalam `.views` berarti modul relatif terhadap package saat ini. Jadi:

- `config.scan('.views')` berarti memindai modul `tutorial.views`
- Jika kita menggunakan `config.scan('views')`, Pyramid akan mencari modul `views` di root Python path
- Menggunakan titik (`.`) memastikan kita memindai modul dalam package yang sama

### 2. Mengapa `assertIn` lebih baik daripada `assertEqual` dalam testing?

**Jawaban:**

`assertIn` lebih baik karena:

1. **Lebih Fleksibel:** `assertIn` hanya memeriksa apakah string tertentu ada dalam response, tidak harus sama persis
2. **Lebih Robust:** Jika format HTML berubah sedikit (spasi, line breaks), test tetap lulus
3. **Lebih Fokus:** Kita hanya menguji konten penting, bukan format keseluruhan
4. **Lebih Mudah:** Tidak perlu menyalin seluruh HTML response

**Contoh:**
```python
# assertIn - lebih fleksibel
self.assertIn(b'Visit', response.body)

# assertEqual - terlalu ketat
self.assertEqual(response.body, b'<body>Visit <a href="/howdy">hello</a></body>')
```

## Troubleshooting

### Error: ModuleNotFoundError: No module named 'tutorial.views'

**Solusi:**
- Pastikan file `tutorial/views.py` ada
- Pastikan `tutorial/__init__.py` ada (untuk membuat `tutorial` menjadi package)
- Pastikan package sudah diinstall dengan `pip install -e .`

### Error: Route 'home' not found

**Solusi:**
- Pastikan `config.add_route()` dipanggil sebelum `config.scan()`
- Pastikan route name dalam `@view_config` sama dengan route name yang didaftarkan

### Error: View function not found

**Solusi:**
- Pastikan `@view_config` decorator berada tepat di atas view function
- Pastikan route name dalam decorator sesuai dengan route yang didaftarkan
- Pastikan `config.scan('.views')` dipanggil setelah routes didaftarkan

## Referensi

- [Pyramid Documentation - Views](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/views.html)
- [Pyramid Documentation - View Configuration](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/viewconfig.html)
- [Pyramid Documentation - Config.scan()](https://docs.pylonsproject.org/projects/pyramid/en/latest/api/config.html#pyramid.config.Configurator.scan)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.
