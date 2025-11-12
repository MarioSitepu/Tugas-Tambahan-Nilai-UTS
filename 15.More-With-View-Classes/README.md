# 15: More With View Classes

## Deskripsi

Proyek ini mendemonstrasikan fitur lanjutan dari view classes dalam Pyramid, termasuk penggunaan view predicates untuk dispatch multiple views berdasarkan request method dan parameter form. Tutorial ini memperlihatkan bagaimana mengorganisir views yang terkait ke dalam satu class dengan berbagi konfigurasi, state, dan logika.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Menggunakan view predicates untuk dispatch multiple views
- Menggunakan `request_method` predicate untuk membedakan GET dan POST
- Menggunakan `request_param` predicate untuk membedakan form submissions
- Berbagi state dan property dalam view class
- Menggunakan `request.route_url()` untuk generate URL secara dinamis

## Background

### View Predicates dalam Pyramid

Pyramid menyediakan **view predicates** yang menentukan view mana yang cocok dengan request berdasarkan berbagai faktor seperti:
- HTTP method (GET, POST, PUT, DELETE, dll)
- Parameter form (field names dalam form)
- Header HTTP
- Dan banyak lagi

View predicates memberikan banyak fleksibilitas dalam menentukan view mana yang akan digunakan untuk request tertentu.

### View Classes untuk Views yang Terkait

Banyak kali views Anda saling terkait: cara berbeda untuk melihat atau bekerja dengan data yang sama, atau REST API yang menangani multiple operations. Mengelompokkan views ini ke dalam view class masuk akal karena:

- **Mengelompokkan views:** Views yang terkait dikelompokkan bersama
- **Memusatkan konfigurasi default:** Menggunakan `@view_defaults` untuk konfigurasi default di level class
- **Berbagi state dan helpers:** State dan helper methods dapat dibagikan antar views dalam class yang sama

### Python Property dalam View Class

Python `@property` memungkinkan kita untuk mendefinisikan computed attributes yang dapat diakses seperti attribute biasa, bukan method. Ini berguna untuk membuat computed values yang tersedia dalam view methods dan templates.

## Tujuan Pembelajaran

1. Memahami konsep view predicates dalam Pyramid
2. Menggunakan `request_method` predicate untuk membedakan GET dan POST requests
3. Menggunakan `request_param` predicate untuk membedakan form submissions
4. Menggunakan Python `@property` dalam view class
5. Menggunakan `request.route_url()` untuk generate URL secara dinamis
6. Berbagi state dan computed values antar views dalam view class

## Struktur Proyek

```
15.More-With-View-Classes/
├── setup.py                 # Setup file dengan dependencies
├── development.ini          # File konfigurasi aplikasi
├── requirements.txt         # Dependencies production
├── tutorial/                # Package aplikasi
│   ├── __init__.py          # Konfigurasi aplikasi dengan route
│   ├── views.py             # View class dengan multiple views dan predicates
│   ├── home.pt              # Template untuk home view
│   ├── hello.pt             # Template untuk hello view dengan form
│   ├── edit.pt              # Template untuk edit view
│   ├── delete.pt            # Template untuk delete view
│   └── tests.py             # Tests untuk view class
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

File `__init__.py` mendefinisikan routes dengan replacement patterns:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy/{first}/{last}')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan:**
- `config.add_route('home', '/')`: Route untuk home view
- `config.add_route('hello', '/howdy/{first}/{last}')`: Route dengan replacement patterns untuk hello view
- `{first}` dan `{last}` adalah placeholder yang akan diekstrak dari URL

### tutorial/views.py

File `views.py` berisi view class dengan multiple views dan view predicates:

```python
from pyramid.view import (
    view_config,
    view_defaults
    )


@view_defaults(route_name='hello')
class TutorialViews:
    def __init__(self, request):
        self.request = request
        self.view_name = 'TutorialViews'

    @property
    def full_name(self):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        return first + ' ' + last

    @view_config(route_name='home', renderer='home.pt')
    def home(self):
        return {'page_title': 'Home View'}

    @view_config(renderer='hello.pt')
    def hello(self):
        return {'page_title': 'Hello View'}

    @view_config(request_method='POST', renderer='edit.pt')
    def edit(self):
        new_name = self.request.params['new_name']
        return {'page_title': 'Edit View', 'new_name': new_name}

    @view_config(request_method='POST', request_param='form.delete',
                 renderer='delete.pt')
    def delete(self):
        print('Deleted')
        return {'page_title': 'Delete View'}
```

**Penjelasan:**
- `@view_defaults(route_name='hello')`: Konfigurasi default untuk semua methods, menggunakan route `hello`
- `self.view_name`: State yang dibagikan antar views
- `@property full_name`: Computed property yang mengakses data dari URL
- `@view_config(route_name='home', renderer='home.pt')`: Override default untuk home view
- `@view_config(renderer='hello.pt')`: Hello view menggunakan default route `hello`
- `@view_config(request_method='POST', renderer='edit.pt')`: Edit view hanya cocok dengan POST requests
- `@view_config(request_method='POST', request_param='form.delete', renderer='delete.pt')`: Delete view hanya cocok dengan POST requests yang memiliki parameter `form.delete`

## Penggunaan

### Menjalankan Aplikasi

1. **Jalankan server development:**

   ```bash
   pserve development.ini --reload
   ```

2. **Akses aplikasi di browser:**

   - Home: http://localhost:6543/
   - Hello: http://localhost:6543/howdy/jane/doe

3. **Uji fungsionalitas:**

   - Klik link di home page untuk menuju hello view
   - Isi form dan klik "Save" untuk melihat edit view
   - Klik "Delete" untuk melihat delete view
   - Perhatikan output di console window

## Penjelasan File

### Template Files

#### tutorial/home.pt

Template untuk home view dengan link ke hello view:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>

<p>Go to the <a href="${request.route_url('hello', first='jane',
        last='doe')}">form</a>.</p>
</body>
</html>
```

**Penjelasan:**
- `${view.view_name}`: Mengakses state dari view class
- `${request.route_url('hello', first='jane', last='doe')}`: Generate URL untuk route `hello` dengan parameter

#### tutorial/hello.pt

Template untuk hello view dengan form:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
<p>Welcome, ${view.full_name}</p>
<form method="POST"
      action="${request.current_route_url()}">
    <input name="new_name"/>
    <input type="submit" name="form.edit" value="Save"/>
    <input type="submit" name="form.delete" value="Delete"/>
</form>
</body>
</html>
```

**Penjelasan:**
- `${view.full_name}`: Mengakses property dari view class (tidak perlu `()` karena adalah property)
- `${request.current_route_url()}`: Generate URL untuk route saat ini
- Form memiliki dua submit buttons dengan nama berbeda (`form.edit` dan `form.delete`)

#### tutorial/edit.pt

Template untuk edit view:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${view.view_name} - ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
<p>You submitted <code>${new_name}</code></p>
</body>
</html>
```

#### tutorial/delete.pt

Template untuk delete view:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${page_title}</title>
</head>
<body>
<h1>${view.view_name} - ${page_title}</h1>
</body>
</html>
```

## Testing

### tutorial/tests.py

File `tests.py` berisi unit tests dan functional tests:

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
        self.assertEqual('Home View', response['page_title'])


class TutorialFunctionalTests(unittest.TestCase):
    def setUp(self):
        from tutorial import main
        app = main({})
        from webtest import TestApp

        self.testapp = TestApp(app)

    def test_home(self):
        res = self.testapp.get('/', status=200)
        self.assertIn(b'TutorialViews - Home View', res.body)
```

### Menjalankan Tests

```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:
```
.. 
2 passed in 0.40 seconds
```

## Analisis

### View Predicates

Dalam tutorial ini, kita melihat bagaimana view predicates digunakan untuk menentukan view mana yang cocok dengan request:

1. **Home View:** Cocok dengan route `home` (GET request ke `/`)
2. **Hello View:** Cocok dengan route `hello` (GET request ke `/howdy/{first}/{last}`)
3. **Edit View:** Cocok dengan route `hello` dan POST request (tanpa parameter khusus)
4. **Delete View:** Cocok dengan route `hello`, POST request, dan parameter `form.delete`

### View Predicate Priority

Ketika multiple views cocok dengan request yang sama, Pyramid menggunakan **predicate priority** untuk menentukan view mana yang akan digunakan. Dalam kasus ini:

- Delete view memiliki predicate yang lebih spesifik (`request_param='form.delete'`) daripada edit view
- Ketika form di-submit dengan button "Delete", delete view akan dipilih
- Ketika form di-submit dengan button "Save", edit view akan dipilih

### Berbagi State dan Property

View class memungkinkan kita untuk berbagi:
- **State:** `self.view_name` yang diassign di `__init__`
- **Computed values:** `@property full_name` yang dihitung dari URL parameters

State dan property ini kemudian tersedia baik dalam view methods maupun templates (misalnya `${view.view_name}` dan `${view.full_name}`).

### URL Generation

Dalam tutorial ini, kita beralih dari hardcoded URLs ke URL generation menggunakan `request.route_url()`:

**Sebelum:**
```html
<a href="/howdy/jane/doe">Howdy</a>
```

**Sesudah:**
```html
<a href="${request.route_url('hello', first='jane', last='doe')}">form</a>
```

**Keuntungan:**
- URLs di-generate secara dinamis berdasarkan route name
- Tidak perlu hardcode URLs
- Lebih fleksibel dan tidak mudah error
- Jika route berubah, URLs otomatis terupdate

## Konsep Penting

### 1. View Predicates

View predicates menentukan view mana yang cocok dengan request berdasarkan berbagai faktor:
- `request_method`: HTTP method (GET, POST, PUT, DELETE, dll)
- `request_param`: Parameter form atau query string
- `header`: HTTP headers
- Dan banyak lagi

### 2. View Predicate Priority

Ketika multiple views cocok dengan request yang sama, Pyramid menggunakan predicate priority:
- Predicates yang lebih spesifik memiliki priority lebih tinggi
- View dengan predicate yang lebih spesifik akan dipilih

### 3. Python Property

Python `@property` memungkinkan kita untuk mendefinisikan computed attributes:
- Dapat diakses seperti attribute biasa (tidak perlu `()`)
- Dihitung setiap kali diakses
- Dapat digunakan dalam view methods dan templates

### 4. URL Generation

Pyramid menyediakan berbagai cara untuk generate URLs:
- `request.route_url()`: Generate absolute URL
- `request.route_path()`: Generate relative path
- `request.current_route_url()`: Generate URL untuk route saat ini

## Extra Credit

### 1. Mengapa template dapat menggunakan `${view.full_name}` tanpa `()`?

**Jawaban:**

`full_name` adalah Python `@property`, bukan method. Property dapat diakses seperti attribute biasa tanpa perlu memanggil dengan `()`. Template engine Chameleon secara otomatis mengenali property dan mengaksesnya sebagai attribute.

**Contoh:**
```python
@property
def full_name(self):
    return first + ' ' + last
```

Dalam template:
```html
${view.full_name}  # Benar - property
${view.full_name()}  # Salah - bukan method
```

### 2. Edit dan delete views keduanya menerima POST requests. Mengapa edit view configuration tidak menangkap POST yang digunakan oleh delete?

**Jawaban:**

Delete view memiliki predicate yang lebih spesifik (`request_param='form.delete'`) daripada edit view. Pyramid menggunakan **predicate priority** untuk menentukan view mana yang cocok:

- Delete view: `request_method='POST'` AND `request_param='form.delete'`
- Edit view: `request_method='POST'` (tanpa predicate tambahan)

Ketika form di-submit dengan button "Delete" (yang memiliki `name="form.delete"`), delete view akan dipilih karena memiliki predicate yang lebih spesifik.

### 3. Kita menggunakan Python `@property` pada `full_name`. Jika kita mereferensinya banyak kali dalam template atau view code, apakah akan dihitung ulang setiap kali? Apakah Pyramid menyediakan sesuatu untuk cache komputasi awal pada property?

**Jawaban:**

Ya, `@property` akan dihitung ulang setiap kali diakses. Jika kita ingin cache hasil komputasi, kita dapat menggunakan beberapa pendekatan:

**Pendekatan 1: Cache di instance variable**
```python
@property
def full_name(self):
    if not hasattr(self, '_full_name'):
        first = self.request.matchdict['first']
        last = self.request.matchdict['last']
        self._full_name = first + ' ' + last
    return self._full_name
```

**Pendekatan 2: Compute di `__init__`**
```python
def __init__(self, request):
    self.request = request
    first = request.matchdict.get('first', '')
    last = request.matchdict.get('last', '')
    self.full_name = first + ' ' + last
```

Pyramid tidak menyediakan mekanisme khusus untuk caching property, jadi kita perlu mengimplementasikannya sendiri jika diperlukan.

### 4. Bisakah kita mengasosiasikan lebih dari satu route dengan view yang sama?

**Jawaban:**

Ya, kita dapat menggunakan multiple `@view_config` decorators pada satu method untuk mengasosiasikannya dengan multiple routes:

```python
@view_config(route_name='home')
@view_config(route_name='home_alt')
def home(self):
    return {'page_title': 'Home View'}
```

Method `home` akan tersedia pada kedua route: `home` dan `home_alt`.

### 5. Ada juga `request.route_path` API. Bagaimana ini berbeda dari `request.route_url`?

**Jawaban:**

- **`request.route_url()`**: Generate **absolute URL** (termasuk scheme, host, dan port)
  - Contoh: `http://localhost:6543/howdy/jane/doe`
  
- **`request.route_path()`**: Generate **relative path** (hanya path, tanpa scheme dan host)
  - Contoh: `/howdy/jane/doe`

**Kapan menggunakan masing-masing:**
- Gunakan `route_url()` untuk links external, emails, atau APIs
- Gunakan `route_path()` untuk links internal dalam aplikasi yang sama (lebih cepat dan tidak bergantung pada host)

## Troubleshooting

### Error: View not found

**Solusi:** Pastikan view predicates sesuai dengan request yang dikirim. Cek request method dan parameters.

### Error: Property tidak dapat diakses di template

**Solusi:** Pastikan property didefinisikan dengan `@property` decorator dan tidak memerlukan parameter.

### Error: URL generation error

**Solusi:** Pastikan semua parameter yang diperlukan untuk route sudah disediakan dalam `route_url()` atau `route_path()`.

## Referensi

- [Pyramid Documentation - View Predicates](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/viewconfig.html#view-predicates)
- [Pyramid Documentation - View Classes](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/views.html#view-classes)
- [Pyramid Documentation - URL Generation](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/urldispatch.html#generating-routes)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.
