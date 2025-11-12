# 05: Unit Tests and pytest

## Deskripsi

Proyek ini mendemonstrasikan cara menulis unit tests untuk aplikasi Pyramid menggunakan pytest. Testing adalah praktik penting dalam pengembangan software untuk memastikan kode bekerja dengan benar dan tetap berfungsi saat dilakukan perubahan di masa depan.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Konsep unit testing
- Penggunaan pytest sebagai test runner
- Pyramid testing helpers untuk memudahkan penulisan tests
- Best practices dalam menulis tests

## Background

### Pentingnya Testing

Sebagai mantra dalam komunitas Python: **"Untested code is broken code."** Testing memastikan:
- Kode bekerja sesuai yang diharapkan
- Perubahan di masa depan tidak merusak fungsionalitas yang sudah ada
- Dokumentasi hidup tentang bagaimana kode seharusnya bekerja
- Kepercayaan diri saat melakukan refactoring

### Pyramid dan Testing

Pyramid memiliki komitmen yang kuat terhadap testing, dengan 100% test coverage sejak pre-release awal. Pyramid menyediakan `pyramid.testing` yang menyediakan helper functions untuk memudahkan penulisan tests.

### pytest vs unittest

Python menyediakan framework testing di standard library (`unittest`), namun pytest menyediakan:
- Sintaks yang lebih sederhana
- Output yang lebih informatif
- Fixtures yang powerful
- Plugin ecosystem yang luas

## Tujuan Pembelajaran

1. Memahami konsep unit testing
2. Menginstall dan menggunakan pytest
3. Menulis unit tests untuk Pyramid views
4. Menggunakan Pyramid testing helpers
5. Menjalankan tests dengan pytest

## Struktur Proyek

```
05.Unit-Tests-and-pytest/
├── setup.py                 # Setup file dengan pytest di dev_requires
├── development.ini          # File konfigurasi aplikasi
├── requirements.txt         # Dependencies production
├── tutorial/                # Package aplikasi
│   ├── __init__.py         # Berisi fungsi main() dan hello_world view
│   └── tests.py            # File unit tests
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
   - Install dependencies development (`pyramid_debugtoolbar`, `pytest`)
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
. 1 passed in 0.14 seconds
```

## Penjelasan File

### setup.py

File ini mendefinisikan package Python dengan pytest di development dependencies:

```python
requires = [
    'pyramid',
    'waitress',
]

dev_requires = [
    'pyramid_debugtoolbar',
    'pytest',
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
- **`pytest`** ditambahkan ke `dev_requires` karena hanya diperlukan untuk development/testing
- Production environment tidak perlu menginstall pytest

### tutorial/tests.py

File ini berisi unit tests untuk aplikasi:

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
```

**Penjelasan:**

1. **`import unittest`**: Menggunakan framework testing standar Python
2. **`from pyramid import testing`**: Import Pyramid testing helpers
3. **`TutorialViewTests`**: Class test yang mewarisi `unittest.TestCase`
4. **`setUp()`**: Method yang dipanggil sebelum setiap test
   - `testing.setUp()`: Menyiapkan testing environment Pyramid
   - Menyimpan config object untuk digunakan di tests (jika diperlukan)
5. **`tearDown()`**: Method yang dipanggil setelah setiap test
   - `testing.tearDown()`: Membersihkan testing environment
6. **`test_hello_world()`**: Test method untuk view `hello_world`
   - Import view di dalam test (bukan di top level) untuk isolasi
   - `testing.DummyRequest()`: Membuat fake HTTP request
   - Memanggil view dengan dummy request
   - `assertEqual()`: Assertion untuk memverifikasi status code adalah 200

**Mengapa import di dalam test method?**

Import di dalam test method memastikan:
- **Isolasi**: Setiap test adalah unit yang independen
- **Avoid side effects**: Import bisa menyebabkan side effects yang mempengaruhi test lain
- **Best practice**: Setiap test harus bisa dijalankan secara independen

**Catatan:** `setUp()` dan `tearDown()` sebenarnya tidak diperlukan di test ini karena kita tidak menggunakan `self.config`. Mereka hanya diperlukan jika test perlu menggunakan config object untuk menambahkan konfigurasi sebelum memanggil view.

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

### Unit Testing

Unit testing adalah praktik menulis tests untuk unit-unit kecil dari kode (biasanya functions atau methods). Setiap test:
- Harus independen (tidak bergantung pada test lain)
- Harus cepat (tidak melakukan operasi yang lambat)
- Harus deterministik (hasil yang sama setiap kali dijalankan)
- Harus menguji satu hal pada satu waktu

### Pyramid Testing Helpers

Pyramid menyediakan `pyramid.testing` yang berisi:
- **`testing.setUp()`**: Menyiapkan testing environment
- **`testing.tearDown()`**: Membersihkan testing environment
- **`testing.DummyRequest()`**: Membuat fake HTTP request untuk testing
- **`testing.DummyResource()`**: Membuat fake resource object
- Dan banyak lagi...

### pytest vs unittest

Kita menggunakan `unittest.TestCase` tapi menjalankan dengan `pytest`. Ini memungkinkan:
- Menggunakan sintaks unittest yang familiar
- Memanfaatkan fitur pytest (output yang lebih baik, fixtures, dll)
- Kompatibilitas dengan kode yang sudah ada

## Menjalankan Aplikasi

Untuk menjalankan aplikasi (bukan tests):

```bash
pserve development.ini --reload
```

Buka browser: http://localhost:6543/

## Extra Credit Questions

### 1. Ubah test untuk assert status code 404

Ubah assertion di `test_hello_world` menjadi:

```python
self.assertEqual(response.status_code, 404)
```

Jalankan pytest lagi:

```bash
pytest tutorial/tests.py -q
```

**Apa yang terjadi?**
- Test akan fail dengan AssertionError
- pytest akan menampilkan error message yang jelas
- Ini menunjukkan bagaimana testing membantu menemukan masalah

### 2. Tambahkan error di view dan lihat hasilnya

Ubah fungsi `hello_world` menjadi:

```python
def hello_world(request):
    return Response(undefined_variable)  # Error: undefined_variable tidak ada
```

Jalankan tests:

```bash
pytest tutorial/tests.py -q
```

**Apa yang terjadi?**
- Test akan fail dengan NameError
- pytest menampilkan traceback yang jelas
- Ini lebih cepat daripada reload browser dan mengeklik refresh

### 3. Test response body

Bagaimana cara menambahkan assertion untuk test HTML value dari response body?

**Jawaban:**

```python
def test_hello_world(self):
    from tutorial import hello_world

    request = testing.DummyRequest()
    response = hello_world(request)

    self.assertEqual(response.status_code, 200)
    self.assertIn(b'Hello World!', response.body)
    # atau
    self.assertEqual(response.body, b'<body><h1>Hello World!</h1></body>')
```

**Penjelasan:**
- `response.body` adalah bytes (bukan string)
- Gunakan `b'...'` untuk bytes literal
- `assertIn()` untuk memeriksa apakah string ada di body
- `assertEqual()` untuk memeriksa exact match

### 4. Mengapa import di dalam test method?

**Jawaban:**

Import di dalam test method memastikan:
- **Isolasi**: Setiap test adalah unit yang independen
- **Avoid side effects**: Import bisa menyebabkan side effects (seperti registrasi di registry, inisialisasi global state, dll) yang mempengaruhi test lain
- **Best practice**: Setiap test harus bisa dijalankan secara independen tanpa bergantung pada urutan eksekusi

Jika kita import di top level:
```python
from tutorial import hello_world  # Di top level

class TutorialViewTests(unittest.TestCase):
    def test_hello_world(self):
        # hello_world sudah diimport
        ...
```

Masalahnya:
- Import di top level dieksekusi saat module dimuat
- Jika ada side effects dari import, mereka akan mempengaruhi semua tests
- Test menjadi tidak independen

## Troubleshooting

### Error: "No module named 'pytest'"

**Solusi:** Pastikan Anda sudah menjalankan `pip install -e ".[dev]"` (dengan extras `[dev]`)

### Error: "No module named 'tutorial'"

**Solusi:** Pastikan package sudah diinstall dengan `pip install -e ".[dev]"`

### Tests tidak ditemukan

**Solusi:** Pastikan file test memiliki prefix `test_` (untuk functions) atau class mewarisi `unittest.TestCase`

### Import error di test

**Solusi:** Pastikan Anda menjalankan pytest dari root directory project, atau gunakan `python -m pytest`

## Referensi

- [Pyramid Documentation - Testing](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/testing.html)
- [pytest Documentation](https://docs.pytest.org/)
- [Python unittest Documentation](https://docs.python.org/3/library/unittest.html)
- [Pyramid Testing API](https://docs.pylonsproject.org/projects/pyramid/en/latest/api/testing.html)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.
