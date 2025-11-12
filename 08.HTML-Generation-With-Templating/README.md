# 08: HTML Generation With Templating

## Deskripsi

Proyek ini mendemonstrasikan cara menggunakan templating system untuk menghasilkan HTML dalam aplikasi Pyramid. Alih-alih menanamkan HTML langsung dalam kode Python, kita menggunakan template engine (Chameleon) untuk memisahkan logika presentasi dari logika bisnis.

Sebagai bagian dari pembelajaran dasar Pyramid, tutorial ini memperkenalkan:
- Menggunakan pyramid_chameleon sebagai template engine
- Membuat template file (.pt) untuk HTML
- Menggunakan renderer dalam @view_config decorator
- Memisahkan data dari presentasi
- Testing yang fokus pada data daripada HTML

## Background

### Mengapa Menggunakan Templating?

**Masalah dengan HTML dalam Kode:**
- HTML yang tertanam dalam string Python sulit dibaca dan dirawat
- Tidak ada syntax highlighting untuk HTML
- Sulit untuk kolaborasi dengan frontend developers
- Tidak ada pemisahan concerns antara logika dan presentasi

**Solusi dengan Templating:**
- HTML berada dalam file template terpisah (.pt)
- Syntax highlighting dan tooling yang lebih baik
- Mudah untuk kolaborasi dengan frontend developers
- Pemisahan yang jelas antara data dan presentasi
- Reusability template yang lebih baik

### Pyramid dan Templating

Pyramid tidak memaksa penggunaan template engine tertentu. Pyramid mendukung berbagai template engine:
- **Chameleon** (ZPT - Zope Page Templates)
- **Jinja2**
- **Mako**

Pyramid memiliki integrasi yang kuat dengan Chameleon melalui `pyramid_chameleon`. Dalam tutorial ini, kita menggunakan Chameleon.

### Chameleon Templates

Chameleon menggunakan syntax Zope Page Templates (ZPT):
- `${variable}` untuk interpolasi variabel
- Mendukung TAL (Template Attribute Language)
- Template files menggunakan ekstensi `.pt`

### Renderers dalam Pyramid

**Renderer** adalah mekanisme Pyramid untuk mengubah return value dari view menjadi response. Ketika kita menggunakan renderer:
1. View function mengembalikan dictionary (data)
2. Pyramid menemukan template berdasarkan renderer yang didefinisikan
3. Template engine menggabungkan data dengan template
4. Hasil HTML dikembalikan sebagai response

## Tujuan Pembelajaran

1. Memahami konsep templating dalam web framework
2. Menginstall dan mengkonfigurasi pyramid_chameleon
3. Membuat template file untuk HTML
4. Menggunakan renderer dalam @view_config decorator
5. Memisahkan data dari presentasi dalam views
6. Mengupdate tests untuk fokus pada data

## Struktur Proyek

```
08.HTML-Generation-With-Templating/
├── setup.py                 # Setup file dengan pyramid_chameleon
├── development.ini          # File konfigurasi dengan reload_templates
├── requirements.txt         # Dependencies production
├── tutorial/                # Package aplikasi
│   ├── __init__.py          # Berisi config.include('pyramid_chameleon')
│   ├── views.py             # Views yang mengembalikan data
│   ├── home.pt              # Template file untuk HTML
│   └── tests.py             # Tests yang fokus pada data
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

### setup.py

File `setup.py` menambahkan `pyramid_chameleon` ke dependencies:

```python
requires = [
    'pyramid',
    'pyramid_chameleon',  # Template engine
    'waitress',
]
```

### tutorial/__init__.py

File `__init__.py` mengaktifkan pyramid_chameleon sebagai renderer:

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.include('pyramid_chameleon')  # Aktifkan Chameleon
    config.add_route('home', '/')
    config.add_route('hello', '/howdy')
    config.scan('.views')
    return config.make_wsgi_app()
```

### development.ini

File `development.ini` menambahkan konfigurasi untuk reload templates secara otomatis:

```ini
[app:main]
use = egg:tutorial
pyramid.reload_templates = true  # Reload templates saat development
pyramid.includes =
    pyramid_debugtoolbar
```

## Penggunaan

### Menjalankan Aplikasi

1. **Jalankan server development:**

   ```bash
   pserve development.ini --reload
   ```

2. **Akses aplikasi di browser:**

   - Home: http://localhost:6543/
   - Hello: http://localhost:6543/howdy

### Views dengan Renderer

Views sekarang mengembalikan dictionary (data) daripada Response object:

```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}
```

**Penjelasan:**
- `renderer='home.pt'` memberitahu Pyramid untuk menggunakan template `home.pt`
- View mengembalikan dictionary dengan data
- Pyramid secara otomatis menggabungkan data dengan template

### Template File (home.pt)

Template menggunakan syntax Chameleon:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Quick Tutorial: ${name}</title>
</head>
<body>
<h1>Hi ${name}</h1>
</body>
</html>
```

**Penjelasan:**
- `${name}` adalah interpolasi variabel dari dictionary yang dikembalikan view
- Template file harus berada di package yang sama dengan views (atau dalam folder templates)

### Menggunakan Template yang Sama untuk Multiple Views

Kita dapat menggunakan template yang sama untuk multiple views dengan data yang berbeda:

```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}

@view_config(route_name='hello', renderer='home.pt')
def hello(request):
    return {'name': 'Hello View'}
```

Kedua views menggunakan template `home.pt` yang sama, tetapi dengan data yang berbeda.

## Testing

### Unit Tests

Unit tests sekarang fokus pada data yang dikembalikan view, bukan HTML:

```python
def test_home(self):
    from .views import home
    
    request = testing.DummyRequest()
    response = home(request)
    # Our view now returns data
    self.assertEqual('Home View', response['name'])
```

**Keuntungan:**
- Tests lebih sederhana dan fokus pada kontrak data
- Tidak perlu mem-parse HTML
- Tests lebih cepat

### Functional Tests

Functional tests masih menguji HTML yang dihasilkan:

```python
def test_home(self):
    res = self.testapp.get('/', status=200)
    self.assertIn(b'<h1>Hi Home View', res.body)
```

**Keuntungan:**
- Menguji integrasi lengkap termasuk templating
- Memastikan template dirender dengan benar
- Menguji output akhir yang diterima user

### Menjalankan Tests

```bash
pytest tutorial/tests.py -q
```

Output yang diharapkan:
```
.... 
4 passed in 0.46 seconds
```

## Analisis

### Perbandingan: Sebelum vs Sesudah Templating

**Sebelum (HTML dalam Kode):**
```python
@view_config(route_name='home')
def home(request):
    return Response('<body>Visit <a href="/howdy">hello</a></body>')
```

**Masalah:**
- HTML tertanam dalam string Python
- Sulit dibaca dan dirawat
- Tidak ada pemisahan concerns

**Sesudah (Templating):**
```python
@view_config(route_name='home', renderer='home.pt')
def home(request):
    return {'name': 'Home View'}
```

**Keuntungan:**
- View fokus pada data/logika
- HTML dalam file template terpisah
- Pemisahan concerns yang jelas
- Mudah untuk kolaborasi

### Testing yang Lebih Baik

**Sebelum:**
```python
def test_home(self):
    response = home(request)
    self.assertEqual(response.status_code, 200)
    self.assertIn(b'Visit', response.body)  # Testing HTML
```

**Sesudah:**
```python
def test_home(self):
    response = home(request)
    self.assertEqual('Home View', response['name'])  # Testing data
```

**Keuntungan:**
- Tests lebih sederhana
- Fokus pada kontrak data
- Tidak tergantung pada format HTML

## Analisis

### Kesimpulan

Dengan menggunakan templating:
1. **Pemisahan Concerns:** Logika bisnis terpisah dari presentasi
2. **Maintainability:** HTML lebih mudah dirawat dalam file template
3. **Collaboration:** Frontend developers dapat bekerja dengan template files
4. **Testing:** Unit tests fokus pada data, bukan HTML
5. **Reusability:** Template dapat digunakan oleh multiple views

Templating adalah best practice dalam pengembangan web modern dan Pyramid menyediakan dukungan yang kuat untuk berbagai template engine.

## Troubleshooting

### Error: Template not found

**Solusi:** Pastikan template file berada di package yang sama dengan views atau dalam folder templates yang dikonfigurasi.

### Error: Variable not found in template

**Solusi:** Pastikan view mengembalikan dictionary dengan key yang sesuai dengan variable yang digunakan di template.

### Error: Template tidak reload

**Solusi:** Pastikan `pyramid.reload_templates = true` di `development.ini` atau restart aplikasi setelah mengubah template.

### Error: Syntax error di template

**Solusi:** Cek syntax Chameleon template. Pastikan menggunakan `${variable}` untuk interpolasi variabel.

## Referensi

- [Pyramid Documentation - Renderers](https://docs.pylonsproject.org/projects/pyramid/en/latest/narr/renderers.html)
- [Chameleon Documentation](https://chameleon.readthedocs.io/)
- [Zope Page Templates](https://zope.readthedocs.io/en/latest/zopebook/ZPT.html)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.
