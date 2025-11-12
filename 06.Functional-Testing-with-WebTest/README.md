# 06: Functional Testing with WebTest

## Deskripsi

Proyek ini mendemonstrasikan cara menulis functional tests (end-to-end full-stack testing) untuk aplikasi Pyramid menggunakan WebTest. Functional testing memungkinkan kita untuk menguji seluruh stack aplikasi, termasuk templating dan seluruh apparatus dari website, bukan hanya unit-unit kecil dari kode.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Konsep functional testing vs unit testing
- Penggunaan WebTest untuk testing full-stack
- Menulis tests yang mensimulasikan HTTP request lengkap
- Testing konten HTML yang dikembalikan

## Background

### Unit Tests vs Functional Tests

**Unit Tests:**
- Menguji unit-unit kecil dari kode secara terisolasi
- Cepat dan fokus pada logika bisnis
- Tidak menguji integrasi dengan komponen lain
- Tidak menguji templating dan rendering HTML

**Functional Tests:**
- Menguji seluruh stack aplikasi (end-to-end)
- Mensimulasikan HTTP request lengkap
- Menguji templating dan rendering HTML
- Menguji integrasi antara komponen
- Tetap cepat karena tidak menggunakan HTTP server sebenarnya

### WebTest

WebTest adalah Python package yang melakukan functional testing. Dengan WebTest:
- Kita dapat menulis tests yang mensimulasikan HTTP request lengkap terhadap WSGI application
- Kita dapat menguji informasi dalam response (status code, headers, body HTML)
- Tests berjalan cepat karena WebTest melewati setup/teardown HTTP server sebenarnya
- Tests cukup cepat untuk menjadi bagian dari TDD (Test-Driven Development)

## Tujuan Pembelajaran

1. Memahami perbedaan unit testing dan functional testing
2. Menginstall dan menggunakan WebTest
3. Menulis functional tests untuk aplikasi Pyramid
4. Menguji konten HTML yang dikembalikan
5. Menggabungkan unit tests dan functional tests dalam satu file test

## Struktur Proyek

```
06.Functional-Testing-with-WebTest/
├── setup.py                 # Setup file dengan webtest di dev_requires
├── development.ini          # File konfigurasi aplikasi
├── requirements.txt         # Dependencies production
├── tutorial/                # Package aplikasi
│   ├── __init__.py         # Berisi fungsi main() dan hello_world view
│   └── tests.py            # File unit tests dan functional tests
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

## Menjalankan Tests

Jalankan tests menggunakan pytest:

```bash
pytest tutorial/tests.py -q
```

**Penjelasan perintah:**
- `pytest`: Test runner
- `tutorial/tests.py`: File yang berisi tests
- `-q`: Quiet mode (output yang lebih ringkas)

**Output yang diharapkan:**
```
.. 2 passed in 0.25 seconds
```

Dua test yang berhasil:
1. Unit test dari `TutorialViewTests`
2. Functional test dari `TutorialFunctionalTests`

## Penjelasan File

### setup.py

File ini mendefinisikan package Python dengan webtest di development dependencies:

```python
requires = [
    'pyramid',
    'waitress',
]

dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
    'webtest',
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
- **`webtest`** ditambahkan ke `dev_requires` karena hanya diperlukan untuk development/testing
- Production environment tidak perlu menginstall webtest

### tutorial/tests.py

File ini berisi unit tests dan functional tests untuk aplikasi:

```python
import unittest

from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_hello_world(self):
        from tutorial import hello_world

        request = testing.DummyRequest()
        response = hello_world(request)
        self.assertEqual(response.status_code, 200)


class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp

        self.testapp = TestApp(app)

    def test_hello_world(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<h1>Hello World!</h1>', res.body)
```

**Penjelasan:**

#### TutorialViewTests (Unit Tests)

Class ini berisi unit tests yang sama seperti tutorial sebelumnya:
- Menguji view secara terisolasi
- Menggunakan `testing.DummyRequest()` untuk membuat fake request
- Menguji status code response

#### TutorialFunctionalTests (Functional Tests)

Class ini berisi functional tests yang menguji seluruh stack:

1. **`setUp()`**: Method yang dipanggil sebelum setiap test
   - `from tutorial import main`: Import fungsi main yang membuat WSGI app
   - `app = main({})`: Membuat WSGI application dengan konfigurasi kosong
   - `from webtest import TestApp`: Import TestApp dari WebTest
   - `self.testapp = TestApp(app)`: Membuat TestApp wrapper untuk WSGI app
     - TestApp memungkinkan kita melakukan HTTP request terhadap app tanpa HTTP server

2. **`test_hello_world()`**: Functional test untuk route `/`
   - `res = self.testapp.get('/', status=200)`: Melakukan GET request ke `/`
     - `status=200`: Assert bahwa status code harus 200 (jika tidak, test akan gagal)
   - `self.assertIn(b'<h1>Hello World!</h1>', res.body)`: Assert bahwa body response mengandung HTML yang diharapkan
     - `b''`: Byte string (lihat penjelasan di bawah)

**Mengapa menggunakan `b''` (byte string)?**

Response body dari WebTest adalah byte string, bukan string biasa. Ini karena:
- HTTP response body secara teknis adalah bytes
- WebTest mengembalikan response body dalam format aslinya (bytes)
- Python 3 membedakan antara string (unicode) dan bytes
- Kita perlu menggunakan byte string (`b''`) untuk membandingkan dengan `res.body`

**Contoh:**
```python
# Benar - menggunakan byte string
self.assertIn(b'<h1>Hello World!</h1>', res.body)

# Salah - akan menghasilkan error
self.assertIn('<h1>Hello World!</h1>', res.body)  # TypeError
```

### tutorial/__init__.py

Berisi fungsi `main()` dan view `hello_world`:

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

Tidak ada perubahan dari tutorial sebelumnya - ini adalah kode yang akan kita test.

## Konsep Penting

### Functional Testing

Functional testing adalah praktik menulis tests yang menguji seluruh stack aplikasi:
- Mensimulasikan HTTP request lengkap
- Menguji routing, views, dan templating
- Menguji response lengkap (status, headers, body)
- Tetap cepat karena tidak menggunakan HTTP server sebenarnya

### WebTest TestApp

`TestApp` dari WebTest:
- Membungkus WSGI application
- Menyediakan methods untuk HTTP requests (get, post, put, delete, dll)
- Mengembalikan response object dengan atribut:
  - `status`: Status code
  - `headers`: Response headers
  - `body`: Response body (byte string)
  - `text`: Response body sebagai string (decoded)

### Unit Tests vs Functional Tests

**Kapan menggunakan Unit Tests:**
- Menguji logika bisnis yang kompleks
- Menguji fungsi-fungsi utility
- Menguji edge cases
- Tests yang sangat cepat

**Kapan menggunakan Functional Tests:**
- Menguji routing dan URL mapping
- Menguji templating dan rendering HTML
- Menguji integrasi antara komponen
- Menguji response lengkap dari aplikasi

**Best Practice:**
- Gunakan kombinasi unit tests dan functional tests
- Unit tests untuk logika bisnis
- Functional tests untuk integrasi dan UI

## Menjalankan Aplikasi

Untuk menjalankan aplikasi (bukan tests):

```bash
pserve development.ini --reload
```

Buka browser: http://localhost:6543/

## Extra Credit

### 1. Mengapa functional tests menggunakan `b''` (byte string)?

**Jawaban:**

Response body dari WebTest adalah byte string karena:
- HTTP response body secara teknis adalah bytes (bukan unicode string)
- WebTest mengembalikan response body dalam format aslinya (bytes)
- Python 3 membedakan antara string (unicode) dan bytes
- Untuk membandingkan dengan `res.body`, kita harus menggunakan byte string

**Contoh:**
```python
# Benar
self.assertIn(b'<h1>Hello World!</h1>', res.body)

# Jika ingin menggunakan string biasa, decode dulu
self.assertIn('<h1>Hello World!</h1>', res.body.decode('utf-8'))
```

### 2. Apa perbedaan antara `res.body` dan `res.text`?

- `res.body`: Byte string (format asli dari HTTP response)
- `res.text`: String (decoded dari bytes menggunakan encoding yang terdeteksi)

**Contoh:**
```python
# Menggunakan body (byte string)
self.assertIn(b'<h1>Hello World!</h1>', res.body)

# Menggunakan text (string)
self.assertIn('<h1>Hello World!</h1>', res.text)
```

### 3. Bagaimana cara menguji POST request?

**Contoh:**
```python
def test_post_request(self):
    res = self.testapp.post('/api/endpoint', 
                            {'key': 'value'},  # Form data
                            status=200)
    self.assertIn(b'success', res.body)
```

## Troubleshooting

### Error: "No module named 'webtest'"

**Solusi:** Pastikan Anda sudah menjalankan `pip install -e ".[dev]"` (dengan extras `[dev]`)

### Error: "No module named 'tutorial'"

**Solusi:** Pastikan package sudah diinstall dengan `pip install -e ".[dev]"`

### Tests tidak ditemukan

**Solusi:** 
- Pastikan file test memiliki prefix `test_` (untuk functions) atau class mewarisi `unittest.TestCase`
- Pastikan file `tests.py` tidak executable (tidak memiliki shebang line `#!/usr/bin/env python`)

### TypeError: a bytes-like object is required

**Solusi:** Pastikan Anda menggunakan byte string (`b''`) saat membandingkan dengan `res.body`

### Import error di test

**Solusi:** Pastikan Anda menjalankan pytest dari root directory project, atau gunakan `python -m pytest`

## Referensi

- [Pyramid Documentation - Testing](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/testing.html)
- [WebTest Documentation](https://docs.pylonsproject.org/projects/webtest/en/latest/)
- [pytest Documentation](https://docs.pytest.org/)
- [Python unittest Documentation](https://docs.python.org/3/library/unittest.html)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.

