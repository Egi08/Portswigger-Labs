
# Blind SQL injection with conditional responses

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a "Welcome back" message in the page if the query returns any rows.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.

Hint: You can assume that the password only contains lowercase, alphanumeric characters.

---------------------------------------------

References: 

- https://portswigger.net/web-security/sql-injection/blind

- https://portswigger.net/web-security/sql-injection/cheat-sheet

---------------------------------------------



Berikut langkah sederhana untuk menyelesaikan lab **"Injeksi SQL buta dengan respons bersyarat"** menggunakan **Burp Suite**:

---

### **Langkah 1: Persiapan**
1. **Buka lab** dan gunakan Burp Suite sebagai proxy.
2. Arahkan browser ke halaman depan aplikasi lab.
3. **Aktifkan intercept** di Burp Suite dan tangkap permintaan HTTP yang berisi cookie `TrackingId`.

---

### **Langkah 2: Verifikasi Kerentanan**
1. **Uji kondisi boolean sederhana:**
   - Modifikasi nilai cookie `TrackingId` menjadi:
     - **`TrackingId=xyz' AND '1'='1`** → Cek apakah muncul pesan "Welcome back".
     - **`TrackingId=xyz' AND '1'='2`** → Cek apakah pesan "Welcome back" hilang.
   - Jika respons sesuai dengan kondisi di atas, aplikasi rentan terhadap injeksi SQL.

2. **Uji keberadaan tabel `users`:**
   - Modifikasi nilai cookie menjadi:
     - **`TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a`**
   - Jika pesan "Welcome back" muncul, tabel `users` ada.

3. **Uji keberadaan pengguna `administrator`:**
   - Modifikasi nilai cookie menjadi:
     - **`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a`**
   - Jika pesan "Welcome back" muncul, pengguna `administrator` ada.

---

### **Langkah 3: Cari Panjang Kata Sandi**
1. Modifikasi cookie untuk memeriksa panjang kata sandi:
   - **`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`**
2. Lanjutkan dengan mengubah angka `1` menjadi `2`, `3`, dan seterusnya hingga pesan "Welcome back" tidak muncul lagi.
3. Catat panjang kata sandi saat kondisi tidak lagi benar (misalnya, 20 karakter).

---

### **Langkah 4: Ekstraksi Karakter Kata Sandi**
1. Kirim permintaan ke **Burp Intruder**:
   - Modifikasi cookie untuk mengeksploitasi posisi pertama kata sandi:
     - **`TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`**
   - Gunakan tanda muatan di sekitar karakter target (`a`):
     - **`TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`**
2. Atur **muatan**:
   - Di tab **Payload**, pilih **Simple list**.
   - Masukkan semua karakter alfanumerik (`a-z`, `0-9`) sebagai muatan.
3. Atur **grep**:
   - Di tab **Grep - Match**, tambahkan kata kunci **"Welcome back"**.
4. Luncurkan serangan untuk menemukan karakter pertama.

5. Ulangi langkah ini untuk setiap posisi karakter:
   - Ubah offset substring, misalnya untuk posisi kedua:
     - **`TrackingId=xyz' AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username='administrator')='§a§`**
   - Lakukan hingga semua karakter password ditemukan.

---
**Cluster Bomb** di Burp Suite Intruder memungkinkan Anda menguji beberapa kombinasi payload di berbagai posisi. Berikut panduan langkah-langkah untuk menggunakannya dalam skenario **SQL Injection** pada cookie `TrackingId`:

---

### **Langkah 1: Kirim Permintaan ke Intruder**
1. **Tangkap permintaan dengan cookie** `TrackingId` di Burp Suite (via Intercept).
2. Klik kanan pada permintaan yang tertangkap dan pilih **"Send to Intruder"**.

---

### **Langkah 2: Atur Target dan Posisi**
1. **Target**:
   - Buka tab **Intruder** → **Target**.
   - Pastikan alamat **Host** dan **Port** sudah benar.

2. **Posisi**:
   - Buka tab **Positions**.
   - Di bagian payload, tambahkan tanda **§** di sekitar variabel atau nilai yang ingin diuji.
   - Contoh:
     ```
     TrackingId=xyz' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§
     ```
     - **§1§**: Offset posisi karakter dalam password.
     - **§2§**: Karakter yang akan diuji.
   - Pastikan mode **Attack type** diatur ke **Cluster bomb**.

---
CLUSTER BOOM

### **Langkah 3: Atur Payload**
1. **Payload Set 1 (Offset posisi)**:
   - Pilih tab **Payloads**.
   - Pada **Payload set** pilih **1**.
   - Masukkan angka **1** hingga panjang password yang telah Anda tentukan.
     - Contoh: Jika panjang password adalah 20, masukkan angka 1–20.

2. **Payload Set 2 (Karakter password)**:
   - Pilih **Payload set** kedua.
   - Tambahkan karakter alfanumerik yang mungkin ada dalam password:
     - `a-z` dan `0-9`.
   - Anda bisa menggunakan menu **Add from list** untuk menambahkannya secara otomatis.

---

### **Langkah 4: Atur Grep**
1. Di tab **Options**, cari bagian **Grep - Match**.
2. Tambahkan pola **"Welcome back"** untuk mendeteksi respons yang sesuai.

---

### **Langkah 5: Luncurkan Cluster Bomb**
1. Klik tombol **Start Attack**.
2. Perhatikan hasil di tab **Intruder**:
   - Cari kombinasi payload di mana kolom **Welcome back** mencentang.
   - Payload di **Set 1** menunjukkan posisi karakter.
   - Payload di **Set 2** menunjukkan nilai karakter pada posisi tersebut.

---

### **Langkah 6: Rekonstruksi Password**
1. Ulangi langkah ini hingga semua posisi password diuji.
2. Gabungkan hasil dari setiap posisi untuk membentuk password lengkap.



### **Langkah 5: Login sebagai Administrator**
1. Gunakan kata sandi yang telah ditemukan.
2. Akses halaman login, masukkan:
   - **Username:** `administrator`
   - **Password:** hasil injeksi SQL.
3. Login untuk menyelesaikan lab.

---

