### **Proof of Concept (PoC) Stored XSS dalam Atribut `onclick` dengan Kurung Sudut dan Tanda Petik Ganda di-HTML-encode serta Tanda Petik Tunggal dan Backslash di-Escape**

#### **Deskripsi Singkat**
Stored Cross-Site Scripting (XSS) adalah jenis kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik disimpan oleh aplikasi web dan direfleksikan kembali kepada pengguna lain. Pada kasus ini, input pengguna disimpan dalam fungsi komentar dan direfleksikan dalam atribut `onclick` sebuah elemen HTML. Namun, karakter kurung sudut (`<`, `>`), tanda petik ganda (`"`), tanda petik tunggal (`'`), dan backslash (`\`) di-encode atau di-escape, sehingga menyulitkan penyisipan kode berbahaya secara langsung. Meskipun demikian, dengan teknik yang tepat, kerentanan ini masih dapat dieksploitasi untuk menyisipkan dan menjalankan kode JavaScript berbahaya.

#### **Langkah-langkah PoC**

1. **Mengirimkan Komentar dengan String Acak pada Input "Website"**
   
   - **Buka Halaman Komentar:** Akses halaman web target yang memungkinkan pengguna untuk memposting komentar.
   - **Isi Formulir Komentar:** Masukkan data berikut ke dalam formulir komentar:
     - **Name:** `test2`
     - **Email:** `test3@test.com`
     - **Website:** `http://test4.com`
     - **Comment:** `test1`
   
   - **Contoh Permintaan HTTP:**
     
     ```
     POST /post/comment HTTP/2
     Host: victim.com
     Content-Type: application/x-www-form-urlencoded
     Content-Length: ...

     csrf=e8yz3UQ62qX7CBfs9PFEanjwdYjzbaMz&postId=1&comment=test1&name=test2&email=test3%40test.com&website=http%3A%2F%2Ftest4.com
     ```
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 1: Mengirimkan Komentar](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/1.png)

2. **Menggunakan Burp Suite untuk Menangkap dan Mengirim ke Repeater**
   
   - **Aktifkan Proxy Burp Suite:**
     - Buka **Burp Suite** dan aktifkan proxy.
     - Pastikan browser Anda dikonfigurasi untuk menggunakan proxy Burp Suite.
   
   - **Kirim Komentar:**
     - Kirim komentar seperti langkah sebelumnya sehingga permintaan HTTP melewati Burp Suite.
   
   - **Intercept Permintaan:**
     - Di tab **Proxy > HTTP history**, temukan permintaan komentar tersebut.
     - Klik kanan pada permintaan dan pilih **"Send to Repeater"**.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 2: Mengirim ke Repeater](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/2.png)

3. **Mengamati Refleksi Input dalam Atribut `onclick`**
   
   - **Kirim Permintaan ke Repeater:**
     - Di tab **Repeater**, perhatikan bagaimana input `http://test4.com` direfleksikan dalam atribut `onclick`:
     
       ```html
       <a id="author" href="http://test4.com" onclick="var tracker={track(){}};tracker.track('http://test4.com');">test2</a>
       ```
   
   - **Analisis Sanitasi:**
     - Tanda petik tunggal (`'`) dan backslash (`\`) di-escape.
     - Karakter kurung sudut (`<`, `>`) dan tanda petik ganda (`"`) di-HTML-encode, misalnya `&lt;`, `&gt;`, dan `&quot;`.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 3: Refleksi dalam Atribut onclick](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/3.png)

4. **Mengirim Payload yang Berhasil Menembus Penyaringan**
   
   - **Susun Payload:**
     - Gunakan payload berikut pada input "Website":
       
       ```
       http://foo?&apos;-alert(1)-&apos;
       ```
     
     - **Penjelasan Payload:**
       - `http://foo?`: URL palsu sebagai basis.
       - `&apos;`: HTML-encode tanda petik tunggal (`'`), berfungsi untuk menutup atribut yang sedang dibuka.
       - `-alert(1)-`: Memanggil fungsi `alert(1)` untuk menampilkan popup.
       - `&apos;`: Menutup kembali atribut atau mencegah sintaks berikutnya.
   
   - **Kirim Komentar dengan Payload:**
     - Ulangi proses pengiriman komentar, tetapi kali ini masukkan payload di atas pada input "Website":
     
       ```
       Website: http://foo?&apos;-alert(1)-&apos;
       ```
   
   - **Contoh Permintaan HTTP dengan Payload:**
     
     ```
     POST /post/comment HTTP/2
     Host: victim.com
     Content-Type: application/x-www-form-urlencoded
     Content-Length: ...

     csrf=e8yz3UQ62qX7CBfs9PFEanjwdYjzbaMz&postId=1&comment=test1&name=test2&email=test3%40test.com&website=http%3A%2F%2Ffoo%3F&apos;-alert(1)-&apos;
     ```
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 4: Mengirim Payload](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/4.png)

5. **Memverifikasi Eksekusi Payload**
   
   - **Salin URL:**
     - Di **Repeater**, klik kanan pada permintaan dengan payload dan pilih **"Copy URL"**.
   
   - **Buka Browser dan Paste URL:**
     - Buka browser (disarankan **Chrome** sesuai instruksi lab) dan tempelkan URL yang telah disalin.
   
   - **Klik Nama Pengguna:**
     - Pada halaman yang ditampilkan, klik pada nama pengguna (`test2`). Jika payload berhasil dieksekusi, sebuah popup alert dengan pesan `1` akan muncul.
   
   - **Gambar Ilustrasi:**
     
     ![Langkah 5: Eksekusi Payload](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/5.png)

#### **Mengapa Payload Ini Berfungsi?**

Meskipun karakter khusus seperti tanda petik tunggal (`'`) dan backslash (`\`) di-escape, serta kurung sudut (`<`, `>`) dan tanda petik ganda (`"`) di-HTML-encode, payload `http://foo?&apos;-alert(1)-&apos;` berhasil memecah atribut `onclick` yang ada. Berikut adalah bagaimana payload bekerja:

1. **Menutup Atribut `onclick`:**
   
   - `&apos;`: Mengakhiri string yang sedang dibuka dalam atribut `onclick`.

2. **Memasukkan Fungsi JavaScript:**
   
   - `-alert(1)-`: Memanggil fungsi `alert(1)` untuk menampilkan popup.

3. **Mencegah Sintaks Berikutnya:**
   
   - `&apos;`: Menutup atribut atau mengomentari sisa kode untuk mencegah error sintaks.

Sehingga, hasil akhir pada HTML setelah injeksi payload adalah sebagai berikut:

```html
<a id="author" href="http://foo?&apos;-alert(1)-&apos;" onclick="var tracker={track(){}};tracker.track('http://foo?&apos;-alert(1)-&apos;');">test2</a>
```

Ketika pengguna mengklik nama `test2`, atribut `onclick` yang telah dimodifikasi akan menjalankan `alert(1)`.

#### **Catatan Penting**

- **Keamanan:** Stored XSS dapat digunakan untuk mencuri informasi sensitif seperti cookie sesi, melakukan pengambilalihan akun, atau mengubah tampilan dan fungsi halaman web.
  
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.

- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)

---

![Gambar Ilustrasi Langkah 1](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/6.png)
![Gambar Ilustrasi Langkah 2](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/7.png)
![Gambar Ilustrasi Langkah 3](images/Stored%20XSS%20into%20onclick%20event%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20and%20backslash%20escaped/8.png)
