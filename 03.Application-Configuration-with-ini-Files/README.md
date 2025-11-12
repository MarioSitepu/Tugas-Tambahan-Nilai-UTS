# 03: Application Configuration with .ini Files

## Deskripsi

Proyek ini mendemonstrasikan penggunaan konfigurasi `.ini` file dengan Pyramid framework. Aplikasi ini menggunakan Pyramid's `pserve` command dengan file konfigurasi `.ini` untuk menjalankan aplikasi dengan cara yang lebih sederhana dan lebih baik.

Pyramid memiliki konsep konfigurasi kelas satu yang terpisah dari kode. Pendekatan ini opsional, tetapi kehadirannya membuat Pyramid berbeda dari framework web Python lainnya. Pyramid memanfaatkan library Setuptools Python, yang menetapkan konvensi untuk menginstal dan menyediakan "entry points" untuk proyek Python.

## Background

### Konfigurasi Terpisah dari Kode

Pyramid memisahkan konfigurasi aplikasi dari kode aplikasi. Ini memungkinkan:
- Mengubah konfigurasi tanpa mengubah kode
- Memiliki multiple konfigurasi untuk environment berbeda (development, production, testing)
- Konfigurasi yang lebih mudah dibaca dan dirawat

### Entry Points

Entry points adalah mekanisme Setuptools yang memungkinkan package Python mendeklarasikan "hook" yang dapat ditemukan dan digunakan oleh aplikasi lain. Pyramid menggunakan entry point untuk mengetahui di mana menemukan WSGI application factory.

### PasteDeploy

Pyramid menggunakan format konfigurasi PasteDeploy (`.ini` file) yang memungkinkan:
- Konfigurasi aplikasi WSGI
- Konfigurasi server WSGI
- Konfigurasi logging Python

## Tujuan Pembelajaran

1. Memahami cara menggunakan `.ini` file untuk konfigurasi Pyramid
2. Memahami konsep entry points dalam Setuptools
3. Menggunakan `pserve` command untuk menjalankan aplikasi
4. Memahami struktur package Pyramid dengan `__init__.py`

## Struktur Proyek

```
03.Application-Configuration-with-ini-Files/
├── setup.py                 # Setup file dengan entry point
├── development.ini          # File konfigurasi aplikasi dan server
├── requirements.txt         # Dependencies proyek
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

3. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

4. **Install package dalam editable mode:**

   ```bash
   pip install -e .
   ```

   Perintah ini akan:
   - Install semua dependencies yang didefinisikan di `setup.py`
   - Install package `tutorial` dalam mode editable (perubahan kode langsung terlihat)
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

   Anda akan melihat halaman dengan teks "Hello World!"

3. **Stop aplikasi:**

   Tekan `Ctrl+C` di terminal

## Penjelasan File

### setup.py

File ini mendefinisikan package Python dengan entry point:

```python
entry_points={
    'paste.app_factory': [
        'main = tutorial:main'
    ],
}
```

- `paste.app_factory`: Tipe entry point untuk WSGI application factory
- `main`: Nama entry point
- `tutorial:main`: Lokasi fungsi `main()` di package `tutorial`

**Mengapa tidak menyebutkan `__init__.py`?**
Ketika Python mengimpor package `tutorial`, secara otomatis mencari dan mengeksekusi `tutorial/__init__.py`. Jadi `tutorial:main` secara otomatis merujuk ke `tutorial/__init__.py:main`.

### development.ini

File konfigurasi yang dibaca oleh `pserve`:

```ini
[app:main]
use = egg:tutorial
```

- `[app:main]`: Section untuk konfigurasi aplikasi WSGI
- `use = egg:tutorial`: Menggunakan entry point `main` dari package `tutorial`

```ini
[server:main]
use = egg:waitress#main
listen = localhost:6543
```

- `[server:main]`: Section untuk konfigurasi server WSGI
- `use = egg:waitress#main`: Menggunakan waitress sebagai WSGI server
- `listen = localhost:6543`: Server akan listen di host `localhost` port `6543`

### tutorial/__init__.py

Berisi fungsi `main()` yang merupakan WSGI application factory:

```python
def main(global_config, **settings):
    config = Configurator(settings=settings)
    config.add_route('hello', '/')
    config.add_view(hello_world, route_name='hello')
    return config.make_wsgi_app()
```

**Parameter fungsi `main()`:**
- `global_config`: Dictionary yang berisi konfigurasi global dari `.ini` file
- `**settings`: Dictionary yang berisi settings dari section `[app:main]` dalam `.ini` file

**Apa itu `**settings`?**
- `**` adalah unpacking operator di Python
- Mengubah dictionary menjadi keyword arguments
- Memungkinkan fungsi menerima settings sebagai named parameters
- Contoh: `{'key': 'value'}` menjadi `key='value'`

**Alur eksekusi:**
1. `pserve` membaca `development.ini`
2. Mencari section `[app:main]` dan menemukan `use = egg:tutorial`
3. `setup.py` mendefinisikan entry point `main = tutorial:main`
4. Pyramid memanggil fungsi `main()` dari `tutorial/__init__.py`
5. Fungsi `main()` membuat dan mengembalikan WSGI application
6. `pserve` menjalankan aplikasi dengan server yang dikonfigurasi di `[server:main]`

## Multiple Configuration Files

Anda dapat membuat multiple `.ini` files untuk environment berbeda:

### development.ini
```ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6543
```

### production.ini
```ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = 0.0.0.0:8080
```

### testing.ini
```ini
[app:main]
use = egg:tutorial

[server:main]
use = egg:waitress#main
listen = localhost:6544
```

Jalankan dengan:
```bash
pserve production.ini
pserve testing.ini --reload
```

## Keuntungan Menggunakan .ini Files

1. **Fleksibilitas**: Ubah konfigurasi tanpa mengubah kode
2. **Environment-specific**: Konfigurasi berbeda untuk dev/prod/test
3. **Separation of Concerns**: Konfigurasi terpisah dari logika aplikasi
4. **Standard Format**: Format `.ini` mudah dibaca dan dipahami
5. **Logging Configuration**: `.ini` file juga digunakan untuk konfigurasi logging Python

## Alternatif: Tanpa .ini Files

Ya, Anda bisa melakukan ini dengan Python code saja tanpa `.ini` file. Namun, menggunakan `.ini` file memberikan:
- Fleksibilitas lebih besar
- Kemudahan maintenance
- Konvensi yang sudah standar di Pyramid
- Kemampuan untuk memiliki multiple konfigurasi

## Troubleshooting

### Error: "No module named 'tutorial'"
**Solusi:** Pastikan Anda sudah menjalankan `pip install -e .`

### Error: "Entry point 'main' not found"
**Solusi:** Pastikan `setup.py` memiliki entry point yang benar dan sudah di-install ulang dengan `pip install -e .`

### Port sudah digunakan
**Solusi:** Ubah port di `development.ini` atau stop aplikasi yang menggunakan port tersebut

### Perubahan kode tidak terlihat
**Solusi:** Pastikan menggunakan flag `--reload` atau restart aplikasi secara manual

## Extra Credit Questions

1. **Jika tidak suka konfigurasi dan/atau file .ini, bisakah dilakukan dengan Python code saja?**
   Ya, bisa. Tapi `.ini` file memberikan fleksibilitas untuk mengubah konfigurasi tanpa mengubah kode.

2. **Bisakah memiliki multiple file .ini untuk satu proyek? Mengapa mungkin ingin melakukan itu?**
   Ya, bisa. Berguna untuk environment berbeda (development, production, testing) dengan konfigurasi berbeda seperti port, database, logging level, dll.

3. **Entry point di setup.py tidak menyebutkan __init__.py saat mendeklarasikan fungsi tutorial:main. Mengapa?**
   Karena Python otomatis mencari di `__init__.py` ketika mengimpor package. `tutorial:main` secara otomatis merujuk ke `tutorial/__init__.py:main`.

4. **Apa tujuan dari **settings? Apa yang ditandakan oleh **?**
   `**settings` adalah unpacking operator yang mengubah dictionary menjadi keyword arguments. Ini memungkinkan fungsi menerima settings dari `.ini` file sebagai named parameters.

## Referensi

- [Pyramid Documentation - Application Configuration](https://docs.pylonsproject.org/projects/pyramid/en/latest/)
- [PasteDeploy Configuration](https://docs.pylonsproject.org/projects/pastedeploy/en/latest/)
- [Setuptools Entry Points](https://setuptools.readthedocs.io/en/latest/userguide/entry_point.html)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran berdasarkan Pyramid Quick Tutorial.

