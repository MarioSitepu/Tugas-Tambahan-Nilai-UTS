# 02: Python Packages for Pyramid Applications

## Deskripsi

Proyek ini adalah langkah kedua dalam tutorial Pyramid, yang mengorganisir aplikasi "Hello World" sebagai Python package minimal di dalam proyek Python minimal. Ini adalah pendekatan yang digunakan oleh sebagian besar pengembangan Python modern.

## Background

### Python Packages

Python developers dapat mengorganisir kumpulan modul dan file menjadi unit bernama yang disebut **package**. Jika sebuah direktori ada di `sys.path` dan memiliki file khusus bernama `__init__.py`, maka direktori tersebut diperlakukan sebagai Python package.

### Python Projects dengan setup.py

Package dapat dibundel, dibuat tersedia untuk instalasi, dan diinstall melalui toolchain yang berorientasi pada file `setup.py`. Untuk tutorial ini, inilah yang perlu Anda ketahui:

1. Kita akan memiliki direktori untuk setiap langkah tutorial sebagai proyek
2. Proyek ini akan berisi `setup.py` yang menyuntikkan fitur project machinery ke dalam direktori
3. Dalam proyek ini kita akan membuat subdirektori `tutorial` menjadi Python package menggunakan file modul Python `__init__.py`
4. Kita akan menjalankan `pip install -e .` untuk menginstall proyek kita dalam development mode

### Development Mode (-e)

Flag `-e` (editable) memungkinkan kita untuk menginstall package dalam "development mode" atau "editable mode". Ini berarti perubahan pada kode sumber akan langsung terlihat tanpa perlu menginstall ulang package.

## Tujuan Pembelajaran

1. Membuat direktori Python "package" dengan `__init__.py`
2. Mendapatkan proyek Python minimal dengan membuat `setup.py`
3. Menginstall proyek tutorial dalam development mode
4. Memahami perbedaan antara single-file application dan packaged application

## Persyaratan

- Python 3.6 atau lebih baru
- pip (Python package manager)
- setuptools (biasanya sudah terinstall dengan pip)

## Struktur Proyek

```
02.Python-Packages-for-Pyramid-Applications/
├── setup.py              # Konfigurasi setuptools untuk package
├── requirements.txt       # Dependencies (opsional, untuk kemudahan)
└── tutorial/             # Python package directory
    ├── __init__.py       # File yang membuat tutorial menjadi package
    └── app.py            # Aplikasi Pyramid
```

## Instalasi

### 1. Install Project dalam Development Mode

Install proyek ini dalam development mode menggunakan pip:

```bash
cd 02.Python-Packages-for-Pyramid-Applications
pip install -e .
```

Atau jika menggunakan virtual environment:

```bash
$VENV/bin/pip install -e .
```

**Apa yang terjadi saat `pip install -e .`?**
- Package `tutorial` diinstall dalam mode editable
- Dependencies (`pyramid` dan `waitress`) diinstall otomatis
- Package dapat diimport dari mana saja dalam environment Python
- Perubahan pada kode langsung terlihat tanpa reinstall

### 2. Verifikasi Instalasi

Verifikasi bahwa package terinstall dengan benar:

```bash
pip list | grep tutorial
```

Atau test import di Python:

```bash
python -c "import tutorial; print(tutorial)"
```

## Menjalankan Aplikasi

### Cara 1: Menjalankan dari Package (Recommended untuk Tutorial)

```bash
python tutorial/app.py
```

Atau dengan virtual environment:

```bash
$VENV/bin/python tutorial/app.py
```

### Cara 2: Menggunakan pserve (Akan dipelajari di step berikutnya)

Setelah setup lebih lanjut, Anda dapat menggunakan:

```bash
pserve development.ini
```

### Akses Aplikasi

Buka browser dan kunjungi:

```
http://localhost:6543/
```

Anda akan melihat halaman dengan teks "Hello World!"

### Menghentikan Aplikasi

Tekan `Ctrl+C` di terminal untuk menghentikan server.

## Penjelasan File

### setup.py

```python
from setuptools import setup

requires = [
    'pyramid',
    'waitress',
]

setup(
    name='tutorial',
    install_requires=requires,
)
```

**Penjelasan:**
- `from setuptools import setup`: Import fungsi setup dari setuptools
- `requires`: List dependencies yang akan diinstall otomatis saat `pip install -e .`
- `setup()`: Konfigurasi package
  - `name='tutorial'`: Nama package
  - `install_requires=requires`: Dependencies yang diperlukan

### tutorial/__init__.py

```python
# package
```

**Penjelasan:**
- File `__init__.py` membuat direktori `tutorial` menjadi Python package
- File ini bisa kosong atau berisi kode inisialisasi package
- Tanpa file ini, Python tidak akan mengenali direktori sebagai package

### tutorial/app.py

```python
from waitress import serve
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')


if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
```

**Penjelasan:**
- Kode aplikasi sama persis dengan step 01
- Sekarang berada di dalam Python package `tutorial`
- Dapat diimport dan digunakan oleh modul lain

## Perbedaan dengan Step 01

| Aspek | Step 01 | Step 02 |
|-------|---------|---------|
| **Struktur** | Single-file | Python package |
| **setup.py** | Tidak ada | Ada |
| **Package** | Tidak ada | Ada (`tutorial/`) |
| **Instalasi** | Langsung run | `pip install -e .` |
| **Import** | Tidak bisa diimport | Bisa diimport |
| **Development Mode** | Tidak | Ya (editable) |

## Konsep Penting

### Python Package

Package adalah cara Python untuk mengorganisir kode menjadi unit yang dapat diimport dan digunakan kembali. Package memungkinkan:
- Organisasi kode yang lebih baik
- Reusability
- Namespace management
- Distribution dan sharing

### Development Mode (-e)

Instalasi dalam development mode berarti:
- Kode sumber tetap di lokasi aslinya
- Perubahan langsung terlihat tanpa reinstall
- Package dapat diimport seperti package yang terinstall normal
- Ideal untuk development

### setup.py dan setuptools

`setup.py` adalah file konfigurasi untuk setuptools yang:
- Mendefinisikan metadata package
- Menentukan dependencies
- Memungkinkan instalasi dan distribusi
- Menyediakan entry points dan scripts

## Analisis

### Keuntungan Menggunakan Package

1. **Organisasi**: Kode terorganisir dengan baik dalam struktur yang jelas
2. **Reusability**: Dapat diimport dan digunakan oleh modul lain
3. **Scalability**: Mudah untuk menambahkan modul dan fitur baru
4. **Distribution**: Dapat didistribusikan dan diinstall oleh orang lain
5. **Development Mode**: Perubahan langsung terlihat tanpa reinstall

### Catatan Penting

**Cara menjalankan aplikasi (`python tutorial/app.py`) adalah sedikit "odd duck"** - kita biasanya tidak akan melakukan ini kecuali untuk tutorial yang mencoba menangkap bagaimana hal-hal ini bekerja langkah demi langkah. Secara umum, **ide yang buruk untuk menjalankan modul Python di dalam package langsung sebagai script**.

Di step-step berikutnya, kita akan belajar cara yang lebih baik untuk menjalankan aplikasi Pyramid.

## Troubleshooting

### Error: `ModuleNotFoundError: No module named 'setuptools'`

**Solusi:**
```bash
pip install setuptools
```

### Error: `ModuleNotFoundError: No module named 'tutorial'`

**Solusi:** Pastikan sudah menginstall package dalam development mode:
```bash
pip install -e .
```

### Error: `Permission denied` saat install

**Solusi:** Gunakan virtual environment atau install dengan `--user`:
```bash
pip install --user -e .
```

### Error: `Address already in use`

**Solusi:** Port 6543 sudah digunakan. Ubah port di `tutorial/app.py`:
```python
serve(app, host='0.0.0.0', port=8080)  # Ganti dengan port lain
```

### Package tidak terupdate setelah perubahan kode

**Solusi:** Pastikan install dengan flag `-e` (editable):
```bash
pip install -e .
```

## Langkah Selanjutnya

Setelah memahami step ini, Anda siap untuk:
- **Step 03**: Application Configuration with .ini Files
- Memahami bagaimana Pyramid menggunakan file konfigurasi
- Belajar menggunakan `pserve` untuk menjalankan aplikasi

## Referensi

- [Python Packaging User Guide](https://packaging.python.org/)
- [setuptools Documentation](https://setuptools.readthedocs.io/)
- [Pyramid Documentation](https://docs.pylonsproject.org/projects/pyramid/en/latest/)
- [PEP 420 - Implicit Namespace Packages](https://peps.python.org/pep-0420/)

## FAQ

### Q: Apakah `__init__.py` masih diperlukan di Python 3?

**A:** Untuk Python 3.3+, `__init__.py` tidak selalu diperlukan untuk namespace packages, tetapi masih diperlukan untuk regular packages dan merupakan praktik yang baik untuk kompatibilitas.

### Q: Apa perbedaan antara `pip install .` dan `pip install -e .`?

**A:** 
- `pip install .`: Install package secara normal (copy ke site-packages)
- `pip install -e .`: Install dalam editable mode (link ke source code)

### Q: Bisakah saya menghapus direktori `tutorial/` setelah install?

**A:** Tidak! Dengan `-e` (editable mode), Python masih membaca dari direktori sumber. Menghapusnya akan menyebabkan error.

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran.

