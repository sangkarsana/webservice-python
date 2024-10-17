# Modul Ajar: Pembuatan Web Service SOAP dengan Python, Spyne, dan SQLite di PythonAnywhere

## Daftar Isi
1. Pendahuluan
2. Konsep Dasar
3. Persiapan Lingkungan
4. Membuat Database SQLite
5. Implementasi Web Service SOAP
6. Deployment di PythonAnywhere
7. Pengujian dengan SoapUI
8. Troubleshooting
9. Latihan dan Proyek

## 1. Pendahuluan

Dalam modul ini, kita akan belajar cara membuat web service SOAP (Simple Object Access Protocol) menggunakan Python dengan library Spyne. Web service ini akan mengelola data buku yang disimpan dalam database SQLite dan di-deploy di platform PythonAnywhere.

### Tujuan Pembelajaran
- Memahami konsep web service SOAP
- Menerapkan CRUD operations menggunakan Python dan SQLite
- Menggunakan Spyne untuk membuat web service SOAP
- Melakukan deployment web service di PythonAnywhere
- Menguji web service menggunakan SoapUI

## 2. Konsep Dasar

### Web Service
Web service adalah sistem perangkat lunak yang dirancang untuk mendukung interoperabilitas mesin-ke-mesin melalui jaringan. SOAP adalah salah satu protokol untuk pertukaran informasi dalam implementasi web service.

### SOAP (Simple Object Access Protocol)
SOAP adalah protokol berbasis XML untuk pertukaran informasi dalam web service. SOAP menggunakan XML untuk format pesannya dan biasanya bergantung pada protokol lain, seperti HTTP, untuk transmisi pesan.

### CRUD Operations
CRUD adalah akronim dari Create, Read, Update, dan Delete, yang merupakan empat fungsi dasar dalam penyimpanan persisten.

### Spyne
Spyne adalah framework Python untuk membangun web service yang mendukung berbagai protokol dan format serialisasi, termasuk SOAP.

### SQLite
SQLite adalah sistem manajemen basis data relasional ringan yang tidak memerlukan server terpisah dan dapat diintegrasikan langsung ke dalam aplikasi.

## 3. Persiapan Lingkungan

1. Buat akun di PythonAnywhere (https://www.pythonanywhere.com/)
2. Login ke dashboard PythonAnywhere
3. Buka konsol Bash

Instal Spyne dengan menjalankan perintah:
```
pip install spyne
```

## 4. Membuat Database SQLite

Buat file `create_db.py`:

```python
import sqlite3

# Koneksi ke database (akan membuat file baru jika belum ada)
conn = sqlite3.connect('bookstore.db')
cursor = conn.cursor()

# Buat tabel books
cursor.execute('''
CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    year INTEGER NOT NULL
)
''')

# Sampel data buku
sample_books = [
    ("To Kill a Mockingbird", "Harper Lee", 1960),
    ("1984", "George Orwell", 1949),
    ("Pride and Prejudice", "Jane Austen", 1813),
    ("The Great Gatsby", "F. Scott Fitzgerald", 1925),
    ("One Hundred Years of Solitude", "Gabriel García Márquez", 1967)
]

# Masukkan sampel data buku
cursor.executemany("INSERT INTO books (title, author, year) VALUES (?, ?, ?)", sample_books)

# Commit perubahan dan tutup koneksi
conn.commit()
conn.close()

print("Database, tabel, dan sampel data berhasil dibuat.")
```

Jalankan script ini dengan perintah:
```
python create_db.py
```

## 5. Implementasi Web Service SOAP

Buat file `soap_book_service.py`:

```python
from spyne import Application, rpc, ServiceBase, Integer, Unicode, Array
from spyne.protocol.soap import Soap11
from spyne.server.wsgi import WsgiApplication
import sqlite3

class DatabaseError(Exception):
    pass

class BookDatabase:
    def __init__(self):
        try:
            self.conn = sqlite3.connect('bookstore.db')
            self.cursor = self.conn.cursor()
        except sqlite3.Error as e:
            raise DatabaseError(f"Database connection failed: {e}")

    def add_book(self, title, author, year):
        try:
            self.cursor.execute("INSERT INTO books (title, author, year) VALUES (?, ?, ?)",
                                (title, author, year))
            self.conn.commit()
            return f"Buku '{title}' berhasil ditambahkan."
        except sqlite3.Error as e:
            raise DatabaseError(f"Error adding book: {e}")

    def get_book(self, book_id):
        try:
            self.cursor.execute("SELECT * FROM books WHERE id = ?", (book_id,))
            book = self.cursor.fetchone()
            if book:
                return f"ID: {book[0]}, Judul: {book[1]}, Penulis: {book[2]}, Tahun: {book[3]}"
            return "Buku tidak ditemukan."
        except sqlite3.Error as e:
            raise DatabaseError(f"Error fetching book: {e}")

    def update_book(self, book_id, title, author, year):
        try:
            self.cursor.execute("UPDATE books SET title = ?, author = ?, year = ? WHERE id = ?",
                                (title, author, year, book_id))
            self.conn.commit()
            if self.cursor.rowcount > 0:
                return f"Buku dengan ID {book_id} berhasil diperbarui."
            return "Buku tidak ditemukan."
        except sqlite3.Error as e:
            raise DatabaseError(f"Error updating book: {e}")

    def delete_book(self, book_id):
        try:
            self.cursor.execute("DELETE FROM books WHERE id = ?", (book_id,))
            self.conn.commit()
            if self.cursor.rowcount > 0:
                return f"Buku dengan ID {book_id} berhasil dihapus."
            return "Buku tidak ditemukan."
        except sqlite3.Error as e:
            raise DatabaseError(f"Error deleting book: {e}")

    def list_books(self):
        try:
            self.cursor.execute("SELECT id, title, author, year FROM books")
            books = self.cursor.fetchall()
            return [f"ID: {book[0]}, Judul: {book[1]}, Penulis: {book[2]}, Tahun: {book[3]}" for book in books]
        except sqlite3.Error as e:
            raise DatabaseError(f"Error fetching books: {e}")

    def __del__(self):
        if hasattr(self, 'conn'):
            self.conn.close()

class BookService(ServiceBase):
    db = BookDatabase()

    @rpc(Unicode, Unicode, Integer, _returns=Unicode)
    def add_book(ctx, title, author, year):
        try:
            return BookService.db.add_book(title, author, year)
        except DatabaseError as e:
            return f"Error: {str(e)}"

    @rpc(Integer, _returns=Unicode)
    def get_book(ctx, book_id):
        try:
            return BookService.db.get_book(book_id)
        except DatabaseError as e:
            return f"Error: {str(e)}"

    @rpc(Integer, Unicode, Unicode, Integer, _returns=Unicode)
    def update_book(ctx, book_id, title, author, year):
        try:
            return BookService.db.update_book(book_id, title, author, year)
        except DatabaseError as e:
            return f"Error: {str(e)}"

    @rpc(Integer, _returns=Unicode)
    def delete_book(ctx, book_id):
        try:
            return BookService.db.delete_book(book_id)
        except DatabaseError as e:
            return f"Error: {str(e)}"

    @rpc(_returns=Array(Unicode))
    def list_books(ctx):
        try:
            return BookService.db.list_books()
        except DatabaseError as e:
            return [f"Error: {str(e)}"]

application = Application([BookService],
    tns='http://bookstore.example.com',
    in_protocol=Soap11(validator='lxml'),
    out_protocol=Soap11()
)

book_service_app = WsgiApplication(application)
```

## 6. Deployment di PythonAnywhere

1. Upload file `soap_book_service.py` dan `bookstore.db` ke direktori home PythonAnywhere Anda.

2. Buat file WSGI (`/var/www/yourusername_pythonanywhere_com_wsgi.py`):

```python
import sys
import os

# Tambahkan path ke direktori proyek Anda
project_home = '/home/yourusername'
if project_home not in sys.path:
    sys.path.insert(0, project_home)

from soap_book_service import book_service_app as application
```

3. Konfigurasi web app di dashboard PythonAnywhere:
   - Pilih Python version (e.g., Python 3.8)
   - Set "Source code" dan "Working directory" ke `/home/yourusername`
   - Set "WSGI configuration file" ke `/var/www/yourusername_pythonanywhere_com_wsgi.py`

4. Klik "Reload" untuk menerapkan perubahan.

## 7. Pengujian dengan SoapUI

1. Unduh dan instal SoapUI (https://www.soapui.org/)
2. Buat proyek baru di SoapUI
3. Masukkan WSDL URL: `http://yourusername.pythonanywhere.com/?wsdl`
4. SoapUI akan membuat request template untuk setiap metode

Contoh pengujian:

### Add Book
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:book="http://bookstore.example.com">
   <soapenv:Header/>
   <soapenv:Body>
      <book:add_book>
         <book:title>The Hobbit</book:title>
         <book:author>J.R.R. Tolkien</book:author>
         <book:year>1937</book:year>
      </book:add_book>
   </soapenv:Body>
</soapenv:Envelope>
```

### Get Book
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:book="http://bookstore.example.com">
   <soapenv:Header/>
   <soapenv:Body>
      <book:get_book>
         <book:book_id>1</book:book_id>
      </book:get_book>
   </soapenv:Body>
</soapenv:Envelope>
```

### List Books
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:book="http://bookstore.example.com">
   <soapenv:Header/>
   <soapenv:Body>
      <book:list_books/>
   </soapenv:Body>
</soapenv:Envelope>
```

## 8. Troubleshooting

Jika mengalami masalah:

1. Periksa log error di PythonAnywhere
2. Pastikan semua file berada di lokasi yang benar
3. Verifikasi koneksi database
4. Periksa izin file dan direktori
5. Pastikan Spyne terinstal dengan benar

## 9. Latihan dan Proyek

1. Tambahkan fitur pencarian buku berdasarkan judul atau penulis.
2. Implementasikan sistem rating untuk buku.
3. Buat antarmuka web sederhana untuk berinteraksi dengan web service.
4. Tambahkan validasi input untuk memastikan data yang dimasukkan valid.
5. Implementasikan sistem autentikasi sederhana untuk web service.

Dengan menyelesaikan modul ini, Anda akan memiliki pemahaman yang kuat tentang cara membuat, men-deploy, dan menguji web service SOAP menggunakan Python, Spyne, dan SQLite di lingkungan PythonAnywhere.
