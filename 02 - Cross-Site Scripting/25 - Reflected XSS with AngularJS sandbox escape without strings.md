### **Proof of Concept (PoC) Reflected XSS dengan Escape Sandbox AngularJS tanpa Menggunakan String**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada lab ini, AngularJS digunakan dengan cara yang tidak biasa di mana fungsi `$eval` tidak tersedia dan penggunaan string dalam AngularJS dibatasi. Tantangan ini adalah melakukan serangan XSS yang dapat melewati sandbox AngularJS dan mengeksekusi fungsi `alert` tanpa menggunakan fungsi `$eval`.

#### **Langkah-langkah PoC**

1. **Memahami Konteks dan Pembatasan AngularJS**

    - **Konteks:** Input pengguna direfleksikan dalam fungsi pencarian blog yang menggunakan AngularJS.
    - **Pembatasan:** Fungsi `$eval` tidak tersedia dan penggunaan string dalam AngularJS dibatasi. Karakter seperti tanda petik tunggal (`'`), tanda petik ganda (`"`), kurung sudut (`<`, `>`), backslash (`\`), dan backticks (`` ` ``) di-encode atau di-escape.

    ![Langkah 1: Konteks AngularJS](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/1.png)

2. **Mengirimkan Permintaan Pencarian dengan String Acak**

    - **Buka Halaman Pencarian:** Akses halaman pencarian pada aplikasi web target.
    - **Isi Kotak Pencarian:** Masukkan string acak alfanumerik, misalnya: `test123`.

    ![Langkah 2: Mengirimkan String Acak](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/2.png)

3. **Menggunakan Burp Suite untuk Menangkap dan Mengirim ke Repeater**

    - **Aktifkan Proxy Burp Suite:**
        - Buka **Burp Suite** dan aktifkan proxy.
        - Pastikan browser Anda dikonfigurasi untuk menggunakan proxy Burp Suite.

    - **Kirim Permintaan Pencarian:**
        - Lakukan pencarian dengan string acak tadi sehingga permintaan (request) dikirim melalui Burp Suite.

    - **Intercept Permintaan:**
        - Di tab **Proxy > HTTP history**, temukan permintaan pencarian tersebut.
        - Klik kanan pada permintaan dan pilih **"Send to Repeater"**.

    ![Langkah 3: Mengirim ke Repeater](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/3.png)

4. **Mengamati Refleksi Input dalam Atribut AngularJS**

    - **Kirim Permintaan ke Repeater:**
        - Di tab **Repeater**, perhatikan bagaimana input `test123` direfleksikan dalam atribut AngularJS:
        
          ```html
          <h1>Hasil Pencarian untuk: test123</h1>
          <script>
              const message = `Hasil pencarian untuk: test123`;
          </script>
          ```
          
        - **Analisis Sanitasi:**
            - Karakter kurung sudut (`<`, `>`), tanda petik ganda (`"`), dan backticks (`` ` ``) di-HTML-encode.
            - Tanda petik tunggal (`'`) dan backslash (`\`) di-escape.

    ![Langkah 4: Refleksi dalam Atribut AngularJS](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/4.png)

5. **Menyusun dan Mengirim Payload untuk Menembus Sanitasi**

    - **Susun Payload:**
        - Payload yang efektif untuk melewati sanitasi tanpa menggunakan string adalah:
        
          ```
          toString().constructor.prototype.charAt= [].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
          ```
        
        - **Penjelasan Payload:**
            - `toString().constructor.prototype.charAt= [].join;`: Mengganti fungsi `charAt` pada prototype `String` dengan fungsi `join` dari array kosong, sehingga setiap pemanggilan `charAt` akan memanggil `join`.
            - `[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1`: Menggunakan filter `orderBy` AngularJS untuk memanggil `fromCharCode` dan mengonversi kode karakter menjadi string `x=alert(1)`.

    - **Kirimkan Payload dalam Permintaan Pencarian:**
        - Ganti parameter `search` pada URL dengan payload di atas. Contohnya:
        
          ```
          https://YOUR-LAB-ID.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
          ```
        
        - **Gambar Ilustrasi:**
          
          ![Langkah 5: Mengirim Payload](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/5.png)

6. **Memverifikasi Eksekusi Payload**

    - **Salin URL:**
        - Di **Repeater**, klik kanan pada permintaan dengan payload dan pilih **"Copy URL"**.

    - **Buka Browser dan Paste URL:**
        - Buka browser (disarankan **Chrome** sesuai instruksi lab) dan tempelkan URL yang telah disalin.

    - **Klik Nama Pengguna atau Area Terkait:**
        - Setelah halaman dimuat, klik pada elemen yang terpengaruh. Jika payload berhasil dieksekusi, fungsi `alert(1)` akan dijalankan, menampilkan popup alert.

    ![Langkah 6: Eksekusi Payload](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/6.png)

#### **Mengapa Payload Ini Berfungsi?**

Meskipun karakter khusus seperti tanda petik tunggal (`'`), backslash (`\`), kurung sudut (`<`, `>`), tanda petik ganda (`"`), dan backticks (`` ` ``) di-encode atau di-escape, payload ini memanfaatkan kemampuan AngularJS untuk memanipulasi fungsi prototype dan filter `orderBy` untuk menyisipkan dan menjalankan kode JavaScript berbahaya. Berikut adalah langkah-langkah yang memungkinkan payload ini berfungsi:

1. **Mengganti Fungsi `charAt`:**
    - Dengan mengganti `String.prototype.charAt` menjadi `[].join`, setiap pemanggilan `charAt` pada string akan menjalankan fungsi `join` dari array kosong. Ini memungkinkan manipulasi lebih lanjut terhadap string dalam AngularJS.

2. **Menggunakan Filter `orderBy`:**
    - Filter `orderBy` digunakan untuk memproses array dengan parameter yang disesuaikan. Dalam payload ini, `toString().constructor.fromCharCode` dipanggil dengan serangkaian kode karakter untuk menghasilkan string `x=alert(1)`.

3. **Menjalankan Fungsi `alert(1)`:**
    - Setelah sandbox AngularJS berhasil di-escape, payload ini menyisipkan dan menjalankan fungsi `alert(1)` sebagai bagian dari eksekusi JavaScript yang valid di dalam halaman web.

#### **Catatan Penting**

- **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif seperti cookie sesi, melakukan pengambilalihan akun, atau mengubah tampilan dan fungsi halaman web.
  
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.

- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)

---

![Gambar Ilustrasi Langkah 1](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/7.png)
![Gambar Ilustrasi Langkah 2](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/8.png)
![Gambar Ilustrasi Langkah 3](images/Reflected%20XSS%20with%20AngularJS%20sandbox%20escape%20without%20strings/9.png)
