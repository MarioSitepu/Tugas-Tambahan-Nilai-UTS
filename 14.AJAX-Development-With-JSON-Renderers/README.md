# 14: AJAX Development With JSON Renderers

## Deskripsi

Proyek ini mendemonstrasikan cara menggunakan JSON renderer di Pyramid untuk mendukung pengembangan aplikasi web modern yang menggunakan AJAX. Dengan JSON renderer, kita dapat mengembalikan data dalam format JSON yang dapat digunakan oleh JavaScript di browser untuk memperbarui UI secara dinamis.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Konsep renderer di Pyramid (tidak hanya untuk HTML)
- Penggunaan JSON renderer untuk mengembalikan data JSON
- Stacking decorator untuk mendukung multiple renderer pada satu view
- Testing response JSON

## Background

### Renderer di Pyramid

Seperti yang kita lihat di **08: HTML Generation With Templating**, deklarasi view dapat menentukan renderer. Output dari view kemudian dijalankan melalui renderer, yang menghasilkan dan mengembalikan response. Kita pertama kali menggunakan Chameleon renderer, kemudian Jinja2 renderer.

**Renderer tidak terbatas pada template yang menghasilkan HTML.** Pyramid menyediakan JSON renderer yang mengambil data Python, melakukan serialisasi ke JSON, dan melakukan beberapa fungsi lain seperti mengatur content type. Bahkan, kita dapat menulis renderer sendiri (atau memperluas renderer built-in) yang berisi logika kustom untuk aplikasi unik kita.

### Aplikasi Web Modern

Aplikasi web modern lebih dari sekadar HTML yang di-render. Halaman dinamis sekarang menggunakan JavaScript untuk memperbarui UI di browser dengan meminta data server sebagai JSON. Pyramid mendukung ini dengan JSON renderer.

## Tujuan Pembelajaran

1. Memahami bahwa renderer tidak terbatas pada template HTML
2. Menggunakan JSON renderer untuk mengembalikan data JSON
3. Stacking decorator untuk mendukung multiple route dan renderer pada satu view
4. Menulis functional tests untuk menguji response JSON
5. Memahami bagaimana view yang data-oriented memudahkan dukungan multiple format output

## Struktur Proyek

```
14.AJAX-Development-With-JSON-Renderers/
├── setup.py                 # Setup file dengan dependencies
├── development.ini          # Konfigurasi aplikasi Pyramid
├── requirements.txt         # Daftar dependencies
├── tutorial/
│   ├── __init__.py          # Konfigurasi aplikasi dengan route hello_json
│   ├── views.py             # View class dengan JSON renderer
│   ├── home.pt              # Template Chameleon
│   └── tests.py             # Unit tests dan functional tests termasuk test_hello_json
└── README.md                # Dokumentasi ini
```

## Instalasi

1. **Buat virtual environment (jika belum ada):**

   **Windows:**
   ```bash
   python -m venv venv
   ```

   **Linux/Mac:**
   ```bash
   python3 -m venv venv
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

   Atau install secara terpisah:

   ```bash
   pip install -e .
   pip install pyramid_debugtoolbar pytest webtest
   ```

## Konfigurasi

### tutorial/__init__.py

File `__init__.py` menambahkan route baru untuk `hello_json`:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.add_route('hello_json', '/howdy.json')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan:**
- `config.add_route('hello_json', '/howdy.json')`: Menambahkan route baru yang memetakan `/howdy.json` ke route name `hello_json`
- Route ini akan menggunakan view yang sama dengan route `hello`, tetapi dengan renderer yang berbeda

### tutorial/views.py

File `views.py` menggunakan stacking decorator untuk mendukung multiple renderer:

```python
from pyramid.view import (
    view_config,
    view_defaults
    )


@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    @view_config(route_name='hello_json', renderer='json')
    def hello(self):
        return {'name': 'Hello View'}
```

**Penjelasan:**
- `@view_config(route_name='hello')`: Decorator pertama untuk route `hello`, menggunakan default renderer `home.pt`
- `@view_config(route_name='hello_json', renderer='json')`: Decorator kedua untuk route `hello_json`, menggunakan JSON renderer
- Kedua decorator "stacked" pada method `hello` yang sama
- Method `hello` mengembalikan dictionary Python yang sama untuk kedua route
- Pyramid akan menggunakan renderer yang sesuai berdasarkan route yang dipanggil

**Stacking Decorator:**
- Kita dapat menambahkan multiple `@view_config` decorator pada satu method
- Setiap decorator dapat memiliki konfigurasi yang berbeda (route, renderer, dll)
- Pyramid akan mendaftarkan method tersebut untuk setiap konfigurasi

### tutorial/tests.py

File `tests.py` menambahkan functional test untuk JSON response:

```python
import unittest

from pyramid import testing


class TutorialViewTests(unittest.TestCase):
    def setUp(self):
        self.config = testing.setUp()

    def tearDown(self):
        testing.tearDown()

    def test_home(self):
        from .views import TutorialViews

        request = testing.DummyRequest()
        inst = TutorialViews(request)
        response = inst.home()
        self.assertEqual('Home View', response['name'])

    def test_hello(self):
        from .views import TutorialViews

        request = testing.DummyRequest()
        inst = TutorialViews(request)
        response = inst.hello()
        self.assertEqual('Hello View', response['name'])


class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp

        self.testapp = TestApp(app)

    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'<h1>Hi Home View', res.body)

    def test_hello(self):
        res = self.testapp.get('/howdy', status=200)
        self.assertIn(b'<h1>Hi Hello View', res.body)

    def test_hello_json(self):
        res = self.testapp.get('/howdy.json', status=200)
        self.assertIn(b'{"name": "Hello View"}', res.body)
        self.assertEqual(res.content_type, 'application/json')
```

**Penjelasan:**
- `test_hello_json`: Functional test baru yang menguji endpoint JSON
- `res = self.testapp.get('/howdy.json', status=200)`: Mengirim GET request ke `/howdy.json`
- `self.assertIn(b'{"name": "Hello View"}', res.body)`: Memverifikasi bahwa response body berisi JSON yang diharapkan
- `self.assertEqual(res.content_type, 'application/json')`: Memverifikasi bahwa content type adalah `application/json`

## Penggunaan

### Menjalankan Tests

Jalankan semua tests:

```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:

```
..... 
5 passed in 0.47 seconds
```

Tests yang dijalankan:
- `test_home` (unit test)
- `test_hello` (unit test)
- `test_home` (functional test)
- `test_hello` (functional test)
- `test_hello_json` (functional test) - **BARU**

### Menjalankan Aplikasi

Jalankan aplikasi Pyramid:

```bash
pserve development.ini --reload
```

Aplikasi akan berjalan di `http://localhost:6543`

### Menguji Endpoint

1. **HTML Response (route `/howdy`):**
   - Buka browser: `http://localhost:6543/howdy`
   - Akan menampilkan HTML: `<h1>Hi Hello View</h1>`

2. **JSON Response (route `/howdy.json`):**
   - Buka browser: `http://localhost:6543/howdy.json`
   - Akan menampilkan JSON: `{"name": "Hello View"}`
   - Content-Type header akan menjadi `application/json`

## Analisis

### View yang Data-Oriented

Sebelumnya kita mengubah view functions dan methods untuk mengembalikan data Python. Perubahan ini ke view layer yang data-oriented membuat penulisan test lebih mudah, memisahkan templating dari logika view.

### JSON Renderer

Karena Pyramid memiliki JSON renderer serta templating renderers, langkah untuk mengembalikan JSON menjadi mudah. Dalam kasus ini, kita mempertahankan view yang sama persis dan mengatur untuk mengembalikan encoding JSON dari data view. Kita melakukan ini dengan:

1. **Menambahkan route** untuk memetakan `/howdy.json` ke route name
2. **Menyediakan `@view_config`** yang mengasosiasikan route name tersebut dengan view yang sudah ada
3. **Mengoverride view defaults** dalam view config yang menyebutkan route `hello_json`, sehingga ketika route cocok, kita menggunakan JSON renderer daripada template renderer `home.pt` yang seharusnya digunakan

### View Predicates untuk AJAX

Untuk aplikasi web murni bergaya AJAX, kita dapat menggunakan kembali route yang ada dengan menggunakan view predicates Pyramid untuk mencocokkan header `Accepts:` yang dikirim oleh implementasi AJAX modern.

### Keterbatasan JSON Renderer

Pyramid's JSON renderer menggunakan base Python JSON encoder, sehingga mewarisi kekuatan dan kelemahannya. Sebagai contoh, Python tidak dapat secara native melakukan JSON encoding pada objek DateTime. Ada sejumlah solusi untuk ini di Pyramid, termasuk memperluas JSON renderer dengan custom renderer.

## Konsep Kunci

### Renderer

- Renderer adalah komponen yang mengambil output dari view dan mengubahnya menjadi response HTTP
- Pyramid memiliki beberapa renderer built-in: Chameleon, Jinja2, JSON, dll
- Kita dapat menulis renderer custom untuk kebutuhan khusus

### Stacking Decorator

- Kita dapat menambahkan multiple `@view_config` decorator pada satu method
- Setiap decorator mendaftarkan method untuk konfigurasi yang berbeda
- Berguna untuk mendukung multiple format output (HTML, JSON, XML, dll) dari view yang sama

### Data-Oriented Views

- Views yang mengembalikan data Python (bukan HTML string) lebih fleksibel
- Data yang sama dapat di-render dengan berbagai renderer
- Memudahkan testing dan pemisahan concerns

## Analisis

### Langkah Selanjutnya

- Pelajari cara membuat custom renderer
- Pelajari cara menggunakan view predicates untuk routing berdasarkan Accept header
- Pelajari cara menangani DateTime dan objek Python lainnya dalam JSON response
- Pelajari cara menggunakan JSON renderer dengan AJAX di frontend

## Troubleshooting

### Error: JSON renderer not found

**Solusi:** JSON renderer adalah built-in renderer Pyramid. Pastikan tidak ada konfigurasi yang salah yang menonaktifkannya.

### Error: Object not JSON serializable

**Solusi:** Pastikan data yang dikembalikan view dapat di-serialize ke JSON. Gunakan dictionary dengan nilai yang dapat di-serialize (string, int, float, list, dict, bool, None).

### Error: Content-Type tidak benar

**Solusi:** JSON renderer secara otomatis mengatur Content-Type menjadi `application/json`. Jika tidak, pastikan tidak ada konfigurasi yang mengoverride ini.

### Error: Stacking decorator tidak bekerja

**Solusi:** Pastikan decorators di-stack dengan benar. Urutan decorator penting - decorator pertama akan didaftarkan terlebih dahulu.

## Referensi

- [Pyramid Renderers Documentation](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/renderers.html)
- [Pyramid JSON Renderer](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/renderers.html#json-renderer)
- [View Configuration](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/viewconfig.html)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.
