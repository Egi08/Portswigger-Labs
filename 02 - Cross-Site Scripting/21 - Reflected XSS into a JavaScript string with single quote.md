### **Proof of Concept (PoC) Reflected XSS pada String JavaScript dengan Tanda Petik Tunggal**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah jenis kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada kasus ini, input pengguna direfleksikan di dalam string JavaScript yang dibungkus dengan tanda petik tunggal (`'`), dan karakter backslash (`\`) di-escape. Hal ini memungkinkan penyerang untuk memecahkan string JavaScript dan menyisipkan kode JavaScript berbahaya.

#### **Langkah-langkah PoC**

1. **Identifikasi Titik Rentan**
   
   Halaman web memiliki fungsi pencarian yang merefleksikan input pengguna dalam elemen HTML `<h1>` dan di dalam variabel JavaScript yang menggunakan tanda petik tunggal.
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/1.png)

2. **Menyusun Payload**
   
   Untuk mengeksploitasi kerentanan ini, kita perlu menyusun payload yang dapat memecah string JavaScript dan menyisipkan skrip berbahaya. Berikut adalah payload yang digunakan:
   
   ```
   ';</script><img src=x onerror=alert(1)><script>var a='a
   ```
   
   **Penjelasan Payload:**
   - `'`: Menutup string JavaScript yang sedang dibuka.
   - `;</script>`: Menutup tag `<script>` yang ada dan memulai tag `<img>` baru.
   - `<img src=x onerror=alert(1)>`: Menyisipkan gambar dengan sumber yang tidak valid sehingga event `onerror` akan dieksekusi, memicu fungsi `alert(1)`.
   - `<script>var a='a`: Membuka kembali tag `<script>` dan mendefinisikan variabel `a` untuk memastikan sintaks JavaScript tetap valid setelah payload.

   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/2.png)

3. **Menggunakan Payload dalam URL**
   
   Masukkan payload ke dalam parameter pencarian pada URL aplikasi web target. Contohnya:
   
   ```
   https://victim.com/search?q=';</script><img src=x onerror=alert(1)><script>var a='a
   ```
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/3.png)

4. **Hasil Akhir pada Halaman Web**
   
   Setelah payload diproses oleh aplikasi web, berikut adalah hasil yang muncul pada halaman:
   
   ```html
   <h1>Hasil Pencarian untuk: '</h1>
   <script>
       var query = ''; </script><img src=x onerror=alert(1)><script>var a='a';
   </script>
   ```
   
   **Penjelasan:**
   - Tag `<h1>` menampilkan input pengguna yang diakhiri dengan tanda petik tunggal.
   - Tag `<script>` awal terputus oleh payload, memungkinkan penyisipan tag `<img>` yang menjalankan `alert(1)` melalui event `onerror`.
   - Tag `<script>` kedua memastikan bahwa variabel JavaScript tetap valid setelah payload.

   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/4.png)

5. **Men-trigger XSS**
   
   Setelah payload dieksekusi, fungsi `alert(1)` akan dipanggil, menampilkan popup alert sebagai bukti bahwa serangan XSS berhasil.
   
   ![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/5.png)

#### **Catatan Penting**
- **Keamanan:** Reflected XSS dapat dimanfaatkan untuk mencuri informasi pengguna, melakukan pengambilalihan sesi, atau mengubah tampilan halaman web.
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.

#### **Referensi:**
- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)

---

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/6.png)
![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/7.png)
![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20single%20quote/8.png)
