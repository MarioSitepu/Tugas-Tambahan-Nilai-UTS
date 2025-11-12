# 11: Dispatching URLs To Views With Routing

## Deskripsi

Proyek ini mendemonstrasikan cara menggunakan routing dengan replacement patterns dalam Pyramid. Routing memungkinkan kita untuk mengekstrak bagian dari URL menjadi dictionary Python yang dapat digunakan dalam view.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Mendefinisikan route dengan replacement patterns (curly braces)
- Menggunakan `request.matchdict` untuk mengakses data dari URL
- Menggunakan data URL dalam view dan template

## Background

### Routing dalam Pyramid

Routing adalah mekanisme untuk mencocokkan pola URL dengan view code. Pyramid memiliki beberapa fitur routing yang berguna:

**Dua Langkah Konfigurasi:**
1. **Setup code** mendaftarkan route name yang digunakan untuk mencocokkan bagian URL
2. **View configuration** mengasosiasikan view dengan route name tersebut

**Mengapa Dua Langkah?**
- Framework Python web lainnya memungkinkan membuat route dan mengasosiasikannya dengan view dalam satu langkah
- Pyramid memungkinkan kita untuk eksplisit dalam urutan routing
- Multiple routes mungkin cocok dengan pola URL yang sama
- Pyramid memberikan fasilitas untuk menghindari masalah ini

### Replacement Patterns

Replacement patterns memungkinkan kita untuk mengekstrak bagian dari URL menjadi dictionary. Pola ini menggunakan curly braces `{}` dalam definisi route.

**Contoh:**
```python
config.add_route('home', '/howdy/{first}/{last}')
```

URL seperti `/howdy/amy/smith` akan:
- Mencocokkan route `home`
- Mengekstrak `amy` ke `first`
- Mengekstrak `smith` ke `last`

Data ini kemudian tersedia dalam `request.matchdict`.

## Tujuan Pembelajaran

1. Memahami konsep routing dengan replacement patterns
2. Mendefinisikan route yang mengekstrak data dari URL
3. Menggunakan `request.matchdict` untuk mengakses data URL
4. Menggunakan data URL dalam view dan template

## Struktur Proyek

```
11.Dispatching-URLs-To-Views-With-Routing/
├── setup.py                 # Setup file
├── development.ini          # File konfigurasi
├── requirements.txt         # Dependencies
├── tutorial/                # Package aplikasi
│   ├── __init__.py          # Konfigurasi aplikasi dengan route
│   ├── views.py             # View yang menggunakan matchdict
│   ├── home.pt              # Template file
│   └── tests.py             # Tests untuk routing
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

   Atau install secara terpisah:

   ```bash
   pip install -e .
   pip install pyramid_debugtoolbar pytest webtest
   ```

## Konfigurasi

### tutorial/__init__.py

File `__init__.py` mendefinisikan route dengan replacement pattern:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/howdy/{first}/{last}')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan:**
- `config.add_route('home', '/howdy/{first}/{last}')`: Mendefinisikan route dengan replacement patterns
- `{first}` dan `{last}` adalah placeholder yang akan diekstrak dari URL
- Nilai-nilai ini akan tersedia dalam `request.matchdict`

### tutorial/views.py

File `views.py` menggunakan `request.matchdict` untuk mengakses data dari URL:

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
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return {
            'name': 'Home View',
            'first': first,
            'last': last
        }
```

**Penjelasan:**
- `self.request.matchdict['first']`: Mengakses nilai `first` dari URL
- `self.request.matchdict['last']`: Mengakses nilai `last` dari URL
- Data ini dikembalikan dalam dictionary untuk digunakan di template

### tutorial/home.pt

Template menggunakan data dari matchdict:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
</head>
<body>
<h1>${name}</h1>
<p>First: ${first}, Last: ${last}</p>
</body>
</html>
```

**Penjelasan:**
- `${first}` dan `${last}` menampilkan nilai dari dictionary yang dikembalikan view
- Template engine akan mengganti placeholder dengan nilai yang sesuai

## Penggunaan

### Menjalankan Aplikasi

1. **Jalankan server development:**

   ```bash
   pserve development.ini --reload
   ```

2. **Akses aplikasi di browser:**

   - URL dengan parameter: http://localhost:6543/howdy/amy/smith
   - URL akan mengekstrak `amy` sebagai `first` dan `smith` sebagai `last`

### Contoh URL

- `/howdy/Jane/Doe` → `first='Jane'`, `last='Doe'`
- `/howdy/John/Smith` → `first='John'`, `last='Smith'`
- `/howdy/amy/smith` → `first='amy'`, `last='smith'`

## Testing

### Unit Tests

Unit tests menguji view dengan mengatur `matchdict` secara manual:

```python
def test_home(self):
    from .views import TutorialViews

    request = testing.DummyRequest()
    request.matchdict['first'] = 'First'
    request.matchdict['last'] = 'Last'
    inst = TutorialViews(request)
    response = inst.home()

    self.assertEqual(response['first'], 'First')
    self.assertEqual(response['last'], 'Last')
```

**Penjelasan:**
- `request.matchdict['first'] = 'First'`: Mengatur nilai `first` secara manual
- `request.matchdict['last'] = 'Last'`: Mengatur nilai `last` secara manual
- Test memverifikasi bahwa view mengembalikan nilai yang benar

### Functional Tests

Functional tests menguji routing melalui HTTP:

```python
def test_home(self):
    res = self.testapp.get('/howdy/Jane/Doe', status=200)
    self.assertIn(b'Jane', res.body)
    self.assertIn(b'Doe', res.body)
```

**Penjelasan:**
- `self.testapp.get('/howdy/Jane/Doe', status=200)`: Mengirim request GET ke URL dengan parameter
- Test memverifikasi bahwa response body mengandung nilai yang diekstrak dari URL

### Menjalankan Tests

```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:
```
..
2 passed in 0.39 seconds
```

## Analisis

### Replacement Patterns

Dalam `__init__.py`, kita melihat perubahan penting dalam deklarasi route:

```python
config.add_route('home', '/howdy/{first}/{last}')
```

Dengan ini, kita memberitahu configurator bahwa URL kita memiliki "replacement pattern". URL seperti `/howdy/amy/smith` akan:
- Mencocokkan route `home`
- Menetapkan `amy` ke `first`
- Menetapkan `smith` ke `last`

### request.matchdict

Kita dapat menggunakan data ini dalam view:

```python
self.request.matchdict['first']
self.request.matchdict['last']
```

`request.matchdict` berisi nilai-nilai dari URL yang cocok dengan "replacement patterns" (curly braces) dalam deklarasi route. Informasi ini kemudian dapat digunakan di mana saja dalam Pyramid yang memiliki akses ke request.

### Alur Kerja

1. **Request datang:** `/howdy/amy/smith`
2. **Routing:** Pyramid mencocokkan URL dengan route `/howdy/{first}/{last}`
3. **Extraction:** Pyramid mengekstrak `amy` → `first`, `smith` → `last`
4. **matchdict:** Nilai-nilai disimpan dalam `request.matchdict`
5. **View:** View mengakses nilai melalui `request.matchdict['first']` dan `request.matchdict['last']`
6. **Template:** Template menampilkan nilai-nilai tersebut

## Extra Credit

**Pertanyaan:** Apa yang terjadi jika Anda mengakses URL `http://localhost:6543/howdy`?

**Jawaban:**
- URL ini tidak akan mencocokkan route `/howdy/{first}/{last}` karena route memerlukan dua parameter
- Pyramid akan mengembalikan 404 Not Found
- Ini adalah hasil yang diharapkan karena route didefinisikan untuk memerlukan kedua parameter

## Konsep Penting

### 1. Replacement Patterns

Replacement patterns menggunakan curly braces `{}` dalam definisi route:

```python
config.add_route('home', '/howdy/{first}/{last}')
```

- `{first}` dan `{last}` adalah placeholder
- Nilai dari URL akan diekstrak ke dictionary dengan key yang sesuai

### 2. request.matchdict

`request.matchdict` adalah dictionary yang berisi nilai-nilai yang diekstrak dari URL:

```python
first = self.request.matchdict['first']
last = self.request.matchdict['last']
```

- Dictionary ini hanya berisi nilai yang cocok dengan replacement patterns
- Dapat diakses di mana saja yang memiliki akses ke request object

### 3. Route Matching

Pyramid mencocokkan URL dengan route berdasarkan pola:

- `/howdy/Jane/Doe` → cocok dengan `/howdy/{first}/{last}`
- `/howdy/Jane` → tidak cocok (kurang satu parameter)
- `/howdy` → tidak cocok (kurang dua parameter)

### 4. URL Parameter Types

Pyramid mendukung berbagai tipe parameter dalam route:

- String (default): `{name}`
- Integer: `{id:\d+}` (hanya angka)
- Custom regex: `{slug:[a-z-]+}` (hanya huruf kecil dan dash)

## Kesimpulan

Dengan menggunakan routing dengan replacement patterns:
1. **Flexibility:** URL dapat mengandung data yang dinamis
2. **Clean URLs:** URL tetap bersih dan readable
3. **Data Extraction:** Data dari URL mudah diakses melalui `matchdict`
4. **Type Safety:** Dapat mendefinisikan tipe parameter untuk validasi

Routing adalah fitur fundamental dalam Pyramid yang memungkinkan kita untuk membuat aplikasi web yang powerful dan fleksibel.

## Referensi

- [Pyramid Documentation - URL Dispatch](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/urldispatch.html)
- [Pyramid Documentation - Route Patterns](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/urldispatch.html#route-patterns)

