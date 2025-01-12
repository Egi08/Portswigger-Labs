
# Reflected XSS in canonical link tag

### **Proof of Concept (PoC) Reflected XSS pada Tag Canonical Link**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) terjadi ketika aplikasi web merefleksikan input pengguna tanpa validasi yang memadai. Pada kasus ini, input pengguna direfleksikan dalam tag `<link rel="canonical">` dan hanya karakter kurung sudut (`<` dan `>`) yang di-escape. Hal ini memungkinkan penyisipan atribut tambahan yang dapat menjalankan kode JavaScript.

#### **Langkah-langkah PoC**

1. **Identifikasi Titik Rentan**
   
   Halaman web memungkinkan pengguna untuk mengirim komentar. Input komentar direfleksikan dalam tag `<link rel="canonical">` tanpa sanitasi yang cukup selain escaping kurung sudut.

   ![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/1.png)

2. **Menyusun Payload**
   
   Untuk menyisipkan atribut `accesskey` dan `onclick`, gunakan payload berikut pada parameter tambahan (`a`):

   ```
   /post?postId=1&a=b'accesskey='X'onclick='alert(1)
   ```

   **Penjelasan Payload:**
   - `postId=1`: Parameter valid yang diperlukan.
   - `a=b'accesskey='X'onclick='alert(1)`: Menambahkan atribut `accesskey` dan `onclick` ke dalam tag `<link>`.

   ![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/2.png)

3. **Menggunakan Payload dalam URL**
   
   Masukkan payload ke dalam URL aplikasi web target:

   ```
   https://victim.com/post?postId=1&a=b'accesskey='X'onclick='alert(1)
   ```

   ![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/3.png)

4. **Hasil pada Tag Canonical**
   
   Setelah payload diproses, tag `<link>` akan menjadi:

   ```html
   <link rel="canonical" accesskey="X" onclick="alert(1)" href="https://example.com/post?postId=1&a=b'accesskey='X'onclick='alert(1)" />
   ```

   ![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/4.png)

5. **Men-trigger XSS**
   
   Tekan salah satu kombinasi tombol berikut di Chrome untuk menjalankan `alert(1)`:
   
   - `ALT+SHIFT+X`
   - `CTRL+ALT+X`
   - `Alt+X`

   Kombinasi tombol ini akan mengaktifkan `accesskey="X"`, memicu event `onclick` yang menjalankan `alert(1)`.

   ![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/5.png)

#### **Catatan Penting**
- **Browser Khusus:** Eksploitasi ini hanya berfungsi di Chrome karena cara penanganan atribut dan event handler pada tag `<link>`.
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan keamanan.

#### **Referensi:**
- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [Penelitian XSS pada Hidden Input Fields](https://portswigger.net/research/xss-in-hidden-input-fields)

---

![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/6.png)
![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/7.png)
![img](images/Reflected%20XSS%20in%20canonical%20link%20tag/8.png)
