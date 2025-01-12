### **Proof of Concept (PoC) Reflected XSS dalam String JavaScript dengan Kurung Sudut dan Tanda Petik Ganda di-HTML-encode serta Tanda Petik Tunggal di-Escape**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada kasus ini, input pengguna direfleksikan di dalam string JavaScript yang dibungkus dengan tanda petik tunggal (`'`), sementara karakter kurung sudut (`<`, `>`) dan tanda petik ganda (`"`) di-HTML-encode, serta tanda petik tunggal di-escape dengan backslash (`\`). Meskipun demikian, kerentanan ini masih dapat dieksploitasi untuk menyisipkan dan menjalankan kode JavaScript berbahaya.

#### **Langkah-langkah PoC**

1. **Mengirimkan String Acak pada Kotak Pencarian**
   
   - Buka halaman pencarian pada aplikasi web target.
   - Masukkan string acak alfanumerik ke dalam kotak pencarian, misalnya: `test123`.
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/1.png)

2. **Menggunakan Burp Suite untuk Menangkap dan Mengirim ke Repeater**
   
   - Buka **Burp Suite** dan aktifkan proxy.
   - Lakukan pencarian dengan string acak tadi sehingga permintaan (request) dikirim melalui Burp Suite.
   - Di tab **Proxy > HTTP history**, temukan permintaan pencarian tersebut.
   - Klik kanan pada permintaan dan pilih **"Send to Repeater"**.
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/2.png)

3. **Mengamati Refleksi Input dalam String JavaScript**
   
   - Di tab **Repeater**, perhatikan bagaimana input `test123` direfleksikan dalam kode JavaScript:
   
     ```html
     <h1>Hasil Pencarian untuk: test123</h1>
     <script>
         var query = 'test123';
     </script>
     ```
     
   - Ini menunjukkan bahwa input pengguna dimasukkan ke dalam string JavaScript dengan tanda petik tunggal, sementara karakter khusus lainnya di-encode atau di-escape.

4. **Mengirim Payload Awal dan Mengamati Penyaringan**
   
   - Coba kirim payload berikut pada parameter pencarian:
   
     ```
     test';alert(1);//
     ```
     
   - Kirimkan permintaan dan amati responsnya:
   
     ```html
     <h1>Hasil Pencarian untuk: test\';alert(1);//</h1>
     <script>
         var query = 'test\';alert(1);//';
     </script>
     ```
     
   - **Hasil:** Tanda petik tunggal (`'`) dalam payload di-escape menjadi `\'`, sehingga mencegah pemutusan string dan eksekusi kode berbahaya.
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/3.png)

5. **Menggunakan Payload yang Berhasil Menembus Penyaringan**
   
   - Ganti input pencarian dengan payload berikut untuk memecahkan string JavaScript dan menyisipkan skrip baru:
   
     ```
     \';alert(1);//
     ```
     
   - Kirimkan permintaan dengan payload tersebut:
   
     ```
     https://victim.com/search?q=\';alert(1);//
     ```
     
   - **Penjelasan Payload:**
     - `\';`: Menutup string JavaScript yang sedang dibuka dan mengakhiri pernyataan sebelumnya.
     - `alert(1);`: Memanggil fungsi `alert` untuk menampilkan popup.
     - `//`: Mengomentari sisa kode JavaScript untuk mencegah error sintaks.
     
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/4.png)

6. **Memverifikasi Eksekusi Payload**
   
   - **Salin URL:** Klik kanan pada permintaan di **Repeater** dan pilih **"Copy URL"**.
   - **Buka Browser:** Tempelkan URL tersebut di browser (disarankan menggunakan Chrome sesuai instruksi lab).
   - **Hasil:** Sebuah popup alert dengan pesan `1` muncul, menandakan bahwa serangan XSS berhasil dieksekusi.
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/5.png)

#### **Mengapa Payload Ini Berfungsi?**

Meskipun tanda petik tunggal (`'`) di-escape dan karakter khusus lainnya di-HTML-encode, payload `\';alert(1);//` berhasil memecah string JavaScript yang ada. Dengan menutup string dan pernyataan sebelumnya, penyerang dapat menyisipkan fungsi `alert(1)` yang dijalankan oleh browser. Bagian `//` mengomentari sisa kode JavaScript untuk mencegah error sintaks, memastikan bahwa payload dapat dieksekusi tanpa gangguan.

#### **Catatan Penting**

- **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif, seperti cookie sesi, atau untuk mengubah tampilan dan fungsi halaman web.
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)

---

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/6.png)
![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/7.png)
![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded%20and%20single%20quotes%20escaped/8.png)
