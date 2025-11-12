# 01: Single-File Web Applications

## Deskripsi

Proyek ini adalah contoh aplikasi web Pyramid yang paling sederhana - sebuah single-file module yang dapat dijalankan langsung oleh Python tanpa perlu package structure, `pip install -e .`, atau konfigurasi tambahan lainnya.

Pyramid adalah framework web Python yang unik karena dapat berfungsi sebagai microframework (aplikasi sederhana dalam satu file) sekaligus dapat diskalakan untuk aplikasi yang sangat besar.

## Background

### Microframework
Microframework adalah istilah marketing, bukan teknis. Mereka memiliki overhead mental yang rendah: mereka melakukan sedikit hal, sehingga Anda hanya perlu fokus pada hal-hal yang Anda buat sendiri.

### Pyramid
Pyramid istimewa karena dapat bertindak sebagai single-file module microframework. Anda dapat memiliki satu file Python yang dapat dieksekusi langsung oleh Python. Tetapi Pyramid juga menyediakan fasilitas untuk diskalakan ke aplikasi terbesar.

### WSGI
Python memiliki standar yang disebut WSGI (Web Server Gateway Interface) yang mendefinisikan bagaimana aplikasi web Python terhubung ke server standar, menerima request yang masuk, dan mengembalikan response. Sebagian besar framework web Python modern mengikuti pola aplikasi "MVC" (model-view-controller), di mana data dalam model memiliki view yang memediasi interaksi dengan sistem luar.

## Tujuan Pembelajaran

1. Mendapatkan aplikasi web Pyramid yang berjalan, sesederhana mungkin
2. Menggunakan ini sebagai dasar yang dipahami dengan baik untuk menambahkan setiap unit kompleksitas
3. Paparan awal terhadap WSGI apps, requests, views, dan responses

## Persyaratan

- Python 3.6 atau lebih baru
- pip (Python package manager)

## Instalasi

1. **Install dependencies:**

   ```bash
   pip install -r requirements.txt
   ```

   Atau install secara manual:

   ```bash
   pip install pyramid waitress
   ```

2. **Verifikasi instalasi:**

   Pastikan semua package terinstall dengan benar:

   ```bash
   pip list | grep -E "(pyramid|waitress)"
   ```

## Menjalankan Aplikasi

1. **Jalankan aplikasi:**

   ```bash
   python app.py
   ```

   Atau jika menggunakan virtual environment:

   ```bash
   $VENV/bin/python app.py
   ```

2. **Akses aplikasi:**

   Buka browser dan kunjungi:

   ```
   http://localhost:6543/
   ```

   Anda akan melihat halaman dengan teks "Hello World!"

3. **Menghentikan aplikasi:**

   Tekan `Ctrl+C` di terminal untuk menghentikan server.

## Struktur Kode

### Penjelasan Baris per Baris

```python
from waitress import serve
from pyramid.config import Configurator
from pyramid.response import Response
```

**Baris 1-3:** Import modul yang diperlukan
- `waitress`: WSGI server untuk menjalankan aplikasi
- `Configurator`: Konfigurator Pyramid untuk mengatur routing dan views
- `Response`: Class untuk membuat HTTP response

```python
def hello_world(request):
    print('Incoming request')
    return Response('<body><h1>Hello World!</h1></body>')
```

**Baris 6-8:** Implementasi view function
- `hello_world(request)`: Fungsi view yang menerima request object
- `print('Incoming request')`: Mencetak log ke console setiap kali ada request
- `return Response(...)`: Mengembalikan HTTP response dengan HTML

```python
if __name__ == '__main__':
    with Configurator() as config:
        config.add_route('hello', '/')
        config.add_view(hello_world, route_name='hello')
        app = config.make_wsgi_app()
    serve(app, host='0.0.0.0', port=6543)
```

**Baris 11-16:** Konfigurasi dan menjalankan aplikasi
- `if __name__ == '__main__':`: Python's way untuk menjalankan kode hanya ketika file dijalankan langsung (bukan ketika di-import)
- `with Configurator() as config:`: Menggunakan context manager untuk konfigurasi
- `config.add_route('hello', '/')`: Menambahkan route dengan nama 'hello' untuk URL '/'
- `config.add_view(hello_world, route_name='hello')`: Menghubungkan view function ke route
- `config.make_wsgi_app()`: Membuat aplikasi WSGI
- `serve(app, host='0.0.0.0', port=6543)`: Menjalankan server di host 0.0.0.0 (semua interface) pada port 6543

## Konsep Penting

### WSGI (Web Server Gateway Interface)
WSGI adalah standar Python yang mendefinisikan interface antara web server dan aplikasi web Python. Ini memungkinkan aplikasi web Python untuk berjalan di berbagai server web.

**WSGI** adalah singkatan dari "Web Server Gateway Interface". Standar ini dimodelkan setelah **CGI (Common Gateway Interface)**, yang merupakan standar web lama untuk menghubungkan server web dengan program eksternal.

### Configurator
Configurator memainkan peran sentral dalam pengembangan Pyramid. Membangun aplikasi dari bagian-bagian yang loosely-coupled melalui Application Configuration adalah ide sentral dalam Pyramid.

### Request-Response Cycle
1. Browser mengirim HTTP request ke server
2. Server (waitress) menerima request
3. WSGI application (Pyramid) memproses request
4. Configurator mencocokkan URL dengan route
5. View function dipanggil dengan request object
6. View function mengembalikan Response
7. Response dikirim kembali ke browser

## Extra Credit - Pertanyaan dan Eksperimen

### 1. Mengapa menggunakan `print('Incoming request')` bukan `print 'Incoming request'`?

`print('Incoming request')` adalah sintaks Python 3, sedangkan `print 'Incoming request'` adalah sintaks Python 2. Python 3 mengharuskan `print()` sebagai function dengan tanda kurung, sementara Python 2 mengizinkan `print` sebagai statement tanpa kurung.

### 2. Apa yang terjadi jika Anda mengembalikan string HTML langsung? Atau sequence of integers?

- **String HTML langsung:** Pyramid akan mengembalikan string sebagai response body, tetapi mungkin tidak memiliki header HTTP yang benar
- **Sequence of integers:** Akan menyebabkan error karena Pyramid Response mengharapkan string atau bytes, bukan integers

### 3. Eksperimen dengan Error Handling

Coba tambahkan kode yang tidak valid di view function, misalnya:

```python
def hello_world(request):
    print(xyz)  # Variable yang tidak didefinisikan
    return Response('<body><h1>Hello World!</h1></body>')
```

1. Matikan aplikasi dengan `Ctrl+C`
2. Restart aplikasi dengan `python app.py`
3. Reload browser
4. Lihat exception di console - ini menunjukkan bagaimana Pyramid menangani error

### 4. WSGI dan CGI

**WSGI** (Web Server Gateway Interface) dimodelkan setelah **CGI** (Common Gateway Interface). CGI adalah standar web yang lebih tua yang memungkinkan server web untuk menjalankan program eksternal untuk menghasilkan konten web dinamis.

## Troubleshooting

### Error: `ModuleNotFoundError: No module named 'waitress'`

**Solusi:** Install dependencies terlebih dahulu:
```bash
pip install -r requirements.txt
```

### Error: `Address already in use`

**Solusi:** Port 6543 sudah digunakan. Ubah port di baris terakhir `app.py`:
```python
serve(app, host='0.0.0.0', port=8080)  # Ganti dengan port lain
```

### Browser tidak bisa mengakses `localhost:6543`

**Solusi:** 
- Pastikan aplikasi sudah berjalan (cek terminal)
- Pastikan tidak ada firewall yang memblokir
- Coba akses `http://127.0.0.1:6543/` sebagai alternatif

## File Structure

```
01.Single-File-Web-Applications/
├── app.py              # Aplikasi Pyramid single-file
├── requirements.txt     # Dependencies yang diperlukan
└── README.md           # Dokumentasi ini
```

## Referensi

- [Pyramid Documentation](https://docs.pylonsproject.org/projects/pyramid/en/latest/)
- [WSGI Specification](https://peps.python.org/pep-3333/)
- [Waitress Documentation](https://docs.pylonsproject.org/projects/waitress/en/latest/)

## Lisensi

Proyek ini dibuat untuk tujuan pembelajaran.

