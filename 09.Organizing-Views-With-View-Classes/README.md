# 09: Organizing Views With View Classes

## Deskripsi

Proyek ini mendemonstrasikan cara mengorganisir views ke dalam view class menggunakan Pyramid. Alih-alih menggunakan view functions yang terpisah, kita mengelompokkan views yang terkait ke dalam sebuah class. Ini memungkinkan kita untuk:

- Mengelompokkan views yang terkait
- Memusatkan konfigurasi default dengan `@view_defaults`
- Berbagi state dan helper methods
- Mengorganisir kode dengan lebih baik

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Mengkonversi view functions menjadi methods pada view class
- Menggunakan `@view_defaults` untuk konfigurasi default di level class
- Mengorganisir views yang terkait dalam satu class
- Mengupdate tests untuk bekerja dengan view class

## Background

### Mengapa Menggunakan View Classes?

**Masalah dengan View Functions:**
- View functions yang terpisah sulit diorganisir ketika jumlahnya banyak
- Konfigurasi yang sama (seperti renderer) harus diulang untuk setiap view
- Tidak ada cara mudah untuk berbagi state atau helper methods antar views
- Views yang terkait secara logis tidak terlihat sebagai satu kesatuan

**Solusi dengan View Classes:**
- Views yang terkait dikelompokkan dalam satu class
- Konfigurasi default dapat didefinisikan di level class dengan `@view_defaults`
- State dan helper methods dapat dibagikan antar views dalam class yang sama
- Kode lebih terorganisir dan mudah dipelihara

### View Classes dalam Pyramid

Pyramid mendukung dua pendekatan untuk views:

**View Functions:**
```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}
```

**View Classes:**
```python
@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request
    
    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}
```

Kedua pendekatan menghasilkan hasil yang sama. Pilihan biasanya berdasarkan kebutuhan organisasi kode.

### @view_defaults Decorator

`@view_defaults` adalah decorator yang diterapkan pada class untuk mendefinisikan konfigurasi default untuk semua methods dalam class tersebut. Konfigurasi ini dapat di-override oleh `@view_config` pada method individual.

**Contoh:**
```python
@view_defaults(renderer='home.pt')  # Default renderer untuk semua methods
class TutorialViews:
    @view_config(route_name='home')  # Menggunakan default renderer
    def home(self):
        return {'name': 'Home View'}
    
    @view_config(route_name='hello', renderer='other.pt')  # Override renderer
    def hello(self):
        return {'name': 'Hello View'}
```

## Tujuan Pembelajaran

1. Memahami konsep view classes dalam Pyramid
2. Mengkonversi view functions menjadi view class
3. Menggunakan `@view_defaults` untuk konfigurasi default
4. Mengorganisir views yang terkait dalam satu class
5. Mengupdate tests untuk bekerja dengan view class

## Struktur Proyek

```
09.Organizing-Views-With-View-Classes/
├── setup.py                 # Setup file
├── development.ini          # File konfigurasi
├── requirements.txt         # Dependencies
├── tutorial/                # Package aplikasi
│   ├── __init__.py          # Konfigurasi aplikasi
│   ├── views.py             # View class dengan methods
│   ├── home.pt              # Template file
│   └── tests.py             # Tests yang diupdate untuk view class
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

File `__init__.py` tetap sama seperti sebelumnya:

```python
from pyramid.config import Configurator


def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

**Penjelasan:**
- `config.scan('.views')` akan menemukan view class dan methods-nya secara otomatis
- Pyramid akan mendaftarkan setiap method yang memiliki `@view_config` decorator

### tutorial/views.py

File `views.py` sekarang menggunakan view class:

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
    def hello(self):
        return {'name': 'Hello View'}
```

**Penjelasan:**
- `@view_defaults(renderer='home.pt')`: Mendefinisikan renderer default untuk semua methods dalam class
- `__init__(self, request)`: Constructor yang menerima request dan menyimpannya sebagai instance variable
- `self.request`: Request object yang dapat diakses oleh semua methods dalam class
- `@view_config(route_name='home')`: Decorator untuk method `home`, menggunakan default renderer
- `@view_config(route_name='hello')`: Decorator untuk method `hello`, menggunakan default renderer

## Penggunaan

### Menjalankan Aplikasi

1. **Jalankan server development:**

   ```bash
   pserve development.ini --reload
   ```

2. **Akses aplikasi di browser:**

   - Home: http://localhost:6543/
   - Hello: http://localhost:6543/howdy

### View Class Pattern

View class mengikuti pola berikut:

1. **Class Definition:**
   - Decorator `@view_defaults` untuk konfigurasi default
   - Constructor `__init__(self, request)` untuk menerima request

2. **View Methods:**
   - Setiap method adalah view yang terpisah
   - Method memiliki `@view_config` decorator
   - Method mengembalikan dictionary dengan data

3. **Shared State:**
   - `self.request` dapat diakses oleh semua methods
   - Helper methods dapat dibagikan antar views

## Testing

### Unit Tests

Unit tests perlu diupdate untuk bekerja dengan view class:

```python
def test_home(self):
    from .views import TutorialViews

    request = testing.DummyRequest()
    inst = TutorialViews(request)  # Buat instance view class
    response = inst.home()  # Panggil method
    self.assertEqual('Home View', response['name'])
```

**Perubahan:**
- Import view class, bukan view function
- Buat instance view class dengan `TutorialViews(request)`
- Panggil method pada instance

### Functional Tests

Functional tests tidak perlu diubah karena mereka menguji melalui HTTP:

```python
def test_home(self):
    res = self.testapp.get('/', status=200)
    self.assertIn(b'<h1>Hi Home View', res.body)
```

**Penjelasan:**
- Functional tests menguji aplikasi melalui HTTP
- Tidak perlu tahu apakah views adalah functions atau methods
- Tetap bekerja dengan baik

### Menjalankan Tests

```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:
```
.... 
4 passed in 0.34 seconds
```

## Analisis

### Perbandingan: View Functions vs View Classes

**Sebelum (View Functions):**
```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}

@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
```

**Masalah:**
- Renderer `'home.pt'` diulang untuk setiap view
- Views yang terkait tidak terlihat sebagai satu kesatuan
- Tidak ada cara untuk berbagi state atau helper methods

**Sesudah (View Classes):**
```python
@view_defaults(renderer='home.pt')
class TutorialViews:
    def __init__(self, request):
        self.request = request

    @view_config(route_name='home')
    def home(self):
        return {'name': 'Home View'}

    @view_config(route_name='hello')
    def hello(self):
        return {'name': 'Hello View'}
```

**Keuntungan:**
- Renderer default didefinisikan sekali di level class
- Views yang terkait dikelompokkan dalam satu class
- State (`self.request`) dapat dibagikan
- Helper methods dapat ditambahkan dan dibagikan

### Testing yang Diupdate

**Sebelum:**
```python
def test_home(self):
    from .views import home
    
    request = testing.DummyRequest()
    response = home(request)  # Langsung panggil function
    self.assertEqual('Home View', response['name'])
```

**Sesudah:**
```python
def test_home(self):
    from .views import TutorialViews
    
    request = testing.DummyRequest()
    inst = TutorialViews(request)  # Buat instance
    response = inst.home()  # Panggil method
    self.assertEqual('Home View', response['name'])
```

**Perubahan:**
- Import class, bukan function
- Buat instance dengan request
- Panggil method pada instance

### Kapan Menggunakan View Classes?

**Gunakan View Classes ketika:**
- Anda memiliki beberapa views yang terkait secara logis
- Views berbagi konfigurasi yang sama (renderer, permission, dll)
- Views perlu berbagi state atau helper methods
- Anda ingin mengorganisir kode dengan lebih baik

**Gunakan View Functions ketika:**
- Views sangat sederhana dan tidak terkait
- Tidak ada konfigurasi yang dibagikan
- Views tidak perlu berbagi state

## Konsep Penting

### 1. @view_defaults Decorator

Decorator ini mendefinisikan konfigurasi default untuk semua methods dalam class:

```python
@view_defaults(renderer='home.pt')
class TutorialViews:
    # Semua methods menggunakan renderer 'home.pt' secara default
```

Konfigurasi default dapat di-override oleh `@view_config` pada method individual.

### 2. View Class Constructor

Constructor `__init__(self, request)` adalah wajib untuk view class:

```python
def __init__(self, request):
    self.request = request
```

- Pyramid akan memanggil constructor dengan request object
- Request disimpan sebagai instance variable untuk digunakan oleh methods

### 3. View Methods

Methods dalam view class adalah views yang terpisah:

```python
@view_config(route_name='home')
def home(self):
    return {'name': 'Home View'}
```

- Setiap method harus memiliki `@view_config` decorator
- Method mengembalikan dictionary (atau response object)
- Method dapat mengakses `self.request`

### 4. Config.scan() dengan View Classes

`config.scan()` bekerja dengan baik untuk view classes:

```python
config.scan('.views')
```

- Pyramid akan menemukan class dengan `@view_defaults`
- Pyramid akan menemukan methods dengan `@view_config`
- Views akan didaftarkan secara otomatis

## Kesimpulan

Dengan menggunakan view classes:
1. **Organisasi:** Views yang terkait dikelompokkan dalam satu class
2. **DRY Principle:** Konfigurasi default tidak perlu diulang
3. **Shared State:** State dan helper methods dapat dibagikan
4. **Maintainability:** Kode lebih mudah dipelihara dan diorganisir

View classes adalah pola yang powerful dalam Pyramid untuk mengorganisir views yang kompleks. Dalam tutorial selanjutnya, kita akan melihat lebih dalam tentang view classes dan fitur-fitur lanjutannya.

## Referensi

- [Pyramid Documentation - View Classes](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/views.html#view-classes)
- [Pyramid Documentation - @view_defaults](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/views.html#view-defaults)

