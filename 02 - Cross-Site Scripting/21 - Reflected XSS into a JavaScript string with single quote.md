### **Proof of Concept (PoC) Reflected XSS dalam String JavaScript dengan Tanda Petik Tunggal**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada kasus ini, input pengguna direfleksikan di dalam string JavaScript yang dibungkus dengan tanda petik tunggal (`'`), dan karakter backslash (`\`) di-escape. Hal ini memungkinkan penyerang untuk memecahkan string JavaScript dan menyisipkan kode JavaScript berbahaya.

#### **Langkah-langkah PoC**

1. **Mengirimkan String Acak pada Kotak Pencarian**

    - Buka halaman pencarian pada aplikasi web target.
    - Masukkan string acak alfanumerik ke dalam kotak pencarian, misalnya: `test123`.
    
    ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/1.png)

2. **Menggunakan Burp Suite untuk Menangkap dan Mengirim ke Repeater**

    - Buka **Burp Suite** dan aktifkan proxy.
    - Lakukan pencarian dengan string acak tadi sehingga permintaan (request) dikirim melalui Burp Suite.
    - Di tab **Proxy > HTTP history**, temukan permintaan pencarian tersebut.
    - Klik kanan pada permintaan dan pilih **"Send to Repeater"**.
    
    ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/2.png)

3. **Mengamati Refleksi Input dalam String JavaScript**

    - Di tab **Repeater**, perhatikan bagaimana input `test123` direfleksikan dalam kode JavaScript:
    
      ```html
      <h1>Hasil Pencarian untuk: test123</h1>
      <script>
          var query = 'test123';
      </script>
      ```
      
    - Ini menunjukkan bahwa input pengguna dimasukkan ke dalam string JavaScript dengan tanda petik tunggal.

4. **Mengirim Payload Awal dan Mengamati Penyaringan**

    - Coba kirim payload berikut pada parameter pencarian:
    
      ```
      test'payload
      ```
      
    - Kirimkan permintaan dan amati responsnya:
    
      ```html
      <h1>Hasil Pencarian untuk: test'payload</h1>
      <script>
          var query = 'test\'payload';
      </script>
      ```
      
    - **Hasil:** Tanda petik tunggal (`'`) dalam payload di-escape menjadi `\'`, sehingga mencegah pemutusan string dan eksekusi kode berbahaya.
    
    ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/3.png)

5. **Menggunakan Payload yang Berhasil Menembus Penyaringan**

    - Ganti input pencarian dengan payload berikut untuk memecahkan string JavaScript dan menyisipkan skrip baru:
    
      ```
      </script><script>alert(1)</script>
      ```
      
    - Kirimkan permintaan dengan payload tersebut:
    
      ```
      https://victim.com/search?q=</script><script>alert(1)</script>
      ```
      
    - **Penjelasan Payload:**
      - `</script>`: Menutup tag `<script>` yang ada.
      - `<script>alert(1)</script>`: Menyisipkan tag `<script>` baru yang menjalankan fungsi `alert(1)`.
      
    ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/4.png)

6. **Memverifikasi Eksekusi Payload**

    - **Salin URL:** Klik kanan pada permintaan di **Repeater** dan pilih **"Copy URL"**.
    - **Buka Browser:** Tempelkan URL tersebut di browser (disarankan menggunakan Chrome sesuai instruksi lab).
    - **Hasil:** Sebuah popup alert dengan pesan `1` muncul, menandakan bahwa serangan XSS berhasil dieksekusi.
    
    ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/5.png)

#### **Mengapa Payload Ini Berfungsi?**

Meskipun karakter tanda petik tunggal (`'`) dan backslash (`\`) di-escape, payload `</script><script>alert(1)</script>` memanfaatkan kemampuan untuk memecah tag `<script>` yang sedang berjalan. Dengan menutup tag `<script>` yang ada dan memulai tag `<script>` baru, penyerang dapat menyisipkan dan menjalankan kode JavaScript yang diinginkan.

#### **Catatan Penting**

- **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif, seperti cookie sesi, atau untuk mengubah tampilan dan fungsi halaman web.
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)

---

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/6.png)
![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/7.png)
![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/8.png)
