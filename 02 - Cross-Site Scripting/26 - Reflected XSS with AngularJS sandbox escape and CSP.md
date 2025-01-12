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
     
     ![Langkah 3: Mengirimkan Eksploitasi]
     ![image](https://github.com/user-attachments/assets/5b8898fd-7eff-4652-a226-bcbe9c131b92)


4. **Memverifikasi Eksekusi Payload**
   
   - **Langkah-langkah:**
     1. **Salin URL Eksploitasi:**
        - Setelah mengirimkan eksploitasi, salin URL yang dihasilkan.
     
     2. **Buka Browser (Chrome):**
        - Buka browser Chrome dan tempelkan URL yang telah disalin.
     
     3. **Klik Elemen yang Terpengaruh:**
        - Setelah halaman dimuat, klik pada nama pengguna atau elemen yang relevan di halaman. Jika payload berhasil dieksekusi, popup alert yang menampilkan `document.cookie` akan muncul.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 4: Eksekusi Payload]
     ![image](https://github.com/user-attachments/assets/d6d9af36-1a1f-4b19-9d42-4911a38708fa)


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
