
# Reflected XSS in canonical link tag
### **Proof of Concept (PoC) dan Penjelasan Query untuk Reflected XSS pada Tag Canonical Link**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah jenis kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada kasus ini, input pengguna direfleksikan dalam tag `<link rel="canonical">` dan hanya karakter kurung sudut (`<` dan `>`) yang di-escape. Hal ini memungkinkan penyerang untuk menyisipkan atribut tambahan yang dapat menjalankan kode JavaScript.

#### **Langkah-langkah PoC**

1. **Memahami Struktur Tag Canonical yang Rentan**

   Tag canonical pada halaman web biasanya terlihat seperti berikut:

   ```html
   <link rel="canonical" href="https://example.com/post?postId=1" />
   ```

   Di sini, parameter `postId` direfleksikan ke atribut `href` tanpa memadai sanitasi selain escaping kurung sudut.

2. **Menyusun Payload untuk Menyisipkan Atribut Berbahaya**

   Untuk mengeksploitasi kerentanan ini, kita dapat menambahkan parameter tambahan yang akan menyisipkan atribut `accesskey` dan `onclick` ke dalam tag `<link>`. Berikut adalah payload yang digunakan:

   ```
   /post?postId=1&a=b'accesskey='X'onclick='alert(1)
   ```

   **Penjelasan Payload:**
   - `postId=1`: Parameter valid yang diperlukan.
   - `a=b'accesskey='X'onclick='alert(1)`: Menambahkan parameter `a` dengan nilai yang menyisipkan dua atribut baru:
     - `accesskey='X'`: Menambahkan akses pintasan keyboard.
     - `onclick='alert(1)'`: Menambahkan event handler yang akan menjalankan fungsi `alert(1)` ketika elemen tersebut diklik atau diaktifkan.

3. **Menggunakan Payload dalam URL**

   Kombinasikan payload tersebut ke dalam URL aplikasi web target. Contohnya:

   ```
   https://victim.com/post?postId=1&a=b'accesskey='X'onclick='alert(1)
   ```

4. **Hasil Akhir pada Tag Canonical**

   Setelah payload diproses oleh aplikasi web, tag `<link>` yang dihasilkan akan menjadi:

   ```html
   <link rel="canonical" accesskey="X" onclick="alert(1)" href="https://example.com/post?postId=1&a=b'accesskey='X'onclick='alert(1)" />
   ```

   **Catatan:** Meskipun tag `<link>` biasanya tidak responsif terhadap event `onclick`, dalam konteks browser tertentu seperti Chrome, atribut ini dapat dieksekusi melalui kombinasi pintasan keyboard yang disediakan.

5. **Men-trigger XSS melalui Kombinasi Tombol**

   Berdasarkan instruksi, pengguna yang terpengaruh dapat menekan kombinasi tombol berikut untuk menjalankan payload:

   - `ALT+SHIFT+X`
   - `CTRL+ALT+X`
   - `Alt+X`

   Kombinasi tombol ini akan mengaktifkan atribut `accesskey="X"`, yang terhubung dengan event `onclick="alert(1)"`, sehingga menghasilkan popup alert.

#### **Mengapa Hanya Berfungsi di Chrome?**

Browser berbeda memiliki cara yang berbeda dalam menangani atribut dan event handler pada elemen HTML. Dalam kasus ini, Chrome mungkin memperbolehkan eksekusi event `onclick` pada tag `<link>` melalui kombinasi akses pintasan keyboard tertentu. Browser lain mungkin tidak mengizinkan atau tidak mengeksekusi event tersebut pada elemen yang biasanya tidak interaktif seperti `<link>`.

#### **Referensi:**
- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [Penelitian XSS pada Hidden Input Fields](https://portswigger.net/research/xss-in-hidden-input-fields)

#### **Gambar Ilustrasi**

*Gambar-gambar yang disediakan dalam deskripsi awal akan membantu memahami proses ini secara visual.*

---

**Catatan Penting:** Eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
