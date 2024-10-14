Modul ajar implementasi web service sederhana dengan SOAP di PythonAnywhere dengan studi kasus **data buku**. Modul ini meliputi pendaftaran akun di PythonAnywhere, pengaturan lingkungan, dan pengembangan web service sederhana menggunakan **Python** dengan pustaka **Spyne**.

---

# **Modul Ajar: Implementasi Web Service Sederhana dengan SOAP di PythonAnywhere**
## **Studi Kasus: Data Buku**

### **Pendahuluan**
Web service memungkinkan aplikasi untuk saling berkomunikasi melalui jaringan. Salah satu metode implementasi web service adalah menggunakan **SOAP** (Simple Object Access Protocol). Pada modul ini, kita akan belajar membuat web service sederhana untuk mengelola data buku menggunakan **Python** di **PythonAnywhere**.

### **Tujuan Pembelajaran**
1. Peserta dapat memahami konsep dasar web service berbasis SOAP.
2. Peserta dapat membuat web service sederhana untuk mengelola data buku.
3. Peserta dapat mempublikasikan web service ke platform cloud **PythonAnywhere**.

---

## **Langkah-Langkah Praktik**

### **Bagian 1: Pendaftaran Akun di PythonAnywhere**

1. **Buka situs PythonAnywhere:**
   - Buka browser dan akses situs [https://www.pythonanywhere.com/](https://www.pythonanywhere.com/).
   
2. **Buat akun:**
   - Klik tombol **Sign up** di kanan atas.
   - Pilih akun **Free** untuk percobaan.
   - Isi detail akun (username, email, dan password), lalu klik **Register**.

3. **Verifikasi akun:**
   - Cek email verifikasi yang dikirim oleh PythonAnywhere, lalu klik tautan untuk memverifikasi.

4. **Login ke PythonAnywhere:**
   - Setelah verifikasi, login dengan akun yang sudah dibuat.

---

### **Bagian 2: Membuat Web Service SOAP**

1. **Membuat Aplikasi Baru:**
   - Setelah login, di dashboard PythonAnywhere, pilih **Web** dari menu di bagian atas.
   - Klik **Add a new web app** untuk membuat aplikasi web baru.
   - Pilih **Manual configuration** dan pilih versi **Python 3.9**.

2. **Mengatur Virtualenv (Opsional):**
   - Jika kamu ingin menggunakan virtual environment, klik **Virtualenv** pada dashboard web dan buat virtual environment baru.
   - Jalankan perintah:
     ```
     mkvirtualenv --python=/usr/bin/python3.9 myenv
     ```
   - Install package yang diperlukan di virtual environment.

3. **Install Pustaka Spyne dan lxml:**
   - Buka terminal di PythonAnywhere dengan memilih menu **Consoles** > **New Console** > **Bash**.
   - Install pustaka **Spyne** dan **lxml** dengan perintah:
     ```
     pip install spyne
     pip install lxml
     ```

4. **Membuat Web Service Data Buku:**
   - Masuk ke direktori proyek di **Files** dan buat file Python baru bernama `soap_buku.py`.
   - Masukkan kode berikut untuk membuat web service SOAP sederhana:
     ```python
     from spyne import Application, rpc, ServiceBase, Unicode, Integer
     from spyne.protocol.soap import Soap11
     from spyne.server.wsgi import WsgiApplication

     # Data buku sederhana
     books = [
         {"id": 1, "title": "Python for Beginners", "author": "John Doe"},
         {"id": 2, "title": "Advanced Python", "author": "Jane Smith"},
     ]

     # Definisikan layanan SOAP
     class BookService(ServiceBase):
         @rpc(Integer, _returns=Unicode)
         def get_book(ctx, book_id):
             for book in books:
                 if book['id'] == book_id:
                     return f"Title: {book['title']}, Author: {book['author']}"
             return "Book not found"

         @rpc(Unicode, Unicode, _returns=Unicode)
         def add_book(ctx, title, author):
             new_id = len(books) + 1
             books.append({"id": new_id, "title": title, "author": author})
             return f"Book '{title}' by {author} added with ID {new_id}"

     # Konfigurasi aplikasi SOAP
     application = Application([BookService], 'spyne.book.soap',
                               in_protocol=Soap11(), out_protocol=Soap11())

     # Menjalankan aplikasi
     wsgi_app = WsgiApplication(application)

     if __name__ == "__main__":
         from wsgiref.simple_server import make_server
         server = make_server('0.0.0.0', 8000, wsgi_app)
         server.serve_forever()
     ```

5. **Menjalankan Aplikasi:**
   - Buka kembali dashboard **Web** dan pada bagian **Code** atur path WSGI file menjadi `soap_buku.py`.
   - Edit file WSGI di `/var/www/yourusername_pythonanywhere_com_wsgi.py` untuk menjalankan aplikasi:
     ```python
     import sys
     sys.path.insert(0, '/home/yourusername/.virtualenvs/myenv/lib/python3.9/site-packages')
     sys.path.insert(0, '/home/yourusername')
    
     from soap_buku import wsgi_app as application
     ```

6. **Reload Aplikasi:**
   - Setelah mengedit WSGI file, kembali ke dashboard **Web**, scroll ke bawah, dan klik **Reload** untuk memulai server.

---

### **Bagian 3: Pengujian Web Service**

1. **Mengakses Endpoint SOAP:**
   - Web service sekarang berjalan di PythonAnywhere. Akses melalui URL:
     ```
     http://yourusername.pythonanywhere.com/
     ```

2. **Menggunakan SOAP Client untuk Uji Coba:**
   - Kamu bisa menggunakan **SOAP UI** atau aplikasi lain untuk mengetes web service ini.
   - Endpoint yang tersedia:
     - `get_book(book_id)` untuk mengambil data buku berdasarkan ID.
     - `add_book(title, author)` untuk menambahkan buku baru.

---

### **Penutup**
Modul ini membahas bagaimana cara membuat dan menerapkan web service SOAP sederhana di PythonAnywhere. Peserta diharapkan memahami dasar SOAP dan bagaimana memublikasikan web service sederhana menggunakan platform cloud.

--- 

