
# Reflected XSS into a JavaScript string with angle brackets HTML encoded
# Reflected XSS ke dalam String JavaScript dengan Tanda Kurung Sudut yang Di-HTML Encode

Lab ini mengandung kerentanan **reflected cross-site scripting (XSS)** pada fungsi pelacakan kueri pencarian di mana tanda kurung sudut di-HTML encode. Refleksi terjadi di dalam string JavaScript. Untuk menyelesaikan lab ini, lakukan serangan cross-site scripting yang keluar dari string JavaScript dan memanggil fungsi `alert`.

---------------------------------------------

Referensi:

- [Contexts dalam Cross-Site Scripting - PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts)

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded/1.png)

---------------------------------------------

Ketika kita mencari kata “aaaa”, halaman dihasilkan dengan kode berikut:

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded/2.png)

Dengan payload seperti:

```
';alert(1);echo 'a
```

Kita melihat bahwa kode sekarang menjadi:

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded/3.png)

Dengan payload ini, fungsi `alert` akan dijalankan:

```
';alert(1)//
```

![img](images/Reflected%20XSS%20into%20a%20JavaScript%20string%20with%20angle%20brackets%20HTML%20encoded/4.png)

### Penjelasan Query

Pada contoh di atas, kita memanfaatkan kerentanan **Reflected XSS** dengan menyuntikkan payload ke dalam string JavaScript yang telah di-HTML encode untuk tanda kurung sudut (`<` dan `>`). Meskipun tanda kurung sudut di-encode, kita masih dapat memecahkan string JavaScript dan menyisipkan kode berbahaya.

**Payload yang digunakan adalah:**

```
';alert(1);echo 'a
```

**Penjelasan Payload:**

1. **'**: Menutup string JavaScript yang sebelumnya.
2. **;alert(1);**: Menambahkan perintah JavaScript untuk memanggil fungsi `alert` dengan argumen `1`.
3. **echo 'a**: Memulai kembali string JavaScript dengan karakter `'a`, memastikan bahwa sintaks HTML tetap valid dan mencegah error yang mungkin menghentikan eksekusi JavaScript.

**Proses Penyisipan dan Eksekusi:**

1. **Menyisipkan Payload:**
   - Saat memasukkan kata pencarian, masukkan payload `';alert(1);echo 'a`.
   - Meski tanda kurung sudut di-encode, penyisipan ini memungkinkan kita untuk keluar dari string JavaScript dan menyisipkan perintah baru.

2. **Kode HTML yang Dihasilkan:**
   - Payload disisipkan ke dalam kode JavaScript seperti berikut:
     ```html
     <script>
         var query = 'aaaa';alert(1);echo 'a';
         // Kode JavaScript lainnya...
     </script>
     ```
   - Dengan payload, kode menjadi:
     ```html
     <script>
         var query = '';alert(1);echo 'a';
         // Kode JavaScript lainnya...
     </script>
     ```

3. **Eksekusi JavaScript:**
   - Setelah payload disisipkan, browser akan menjalankan perintah `alert(1)` yang ditambahkan, menampilkan kotak dialog alert dengan pesan `1`.

**Mengapa Payload Berhasil:**

- **Pemecahan String JavaScript:** Dengan memasukkan karakter `'`, kita dapat keluar dari string JavaScript yang ada dan menambahkan perintah baru.
- **Sintaks JavaScript yang Valid:** Payload dirancang sedemikian rupa sehingga setelah penyisipan, sintaks JavaScript tetap valid, memungkinkan eksekusi perintah tambahan.
- **Reflected XSS:** Payload dikirim ke server dan langsung direfleksikan kembali ke pengguna tanpa disanitasi yang memadai, memungkinkan eksekusi kode JavaScript.

**Langkah-langkah Penyelesaian Lab:**

1. **Kirim Pencarian:**
   - Masukkan payload `';alert(1);echo 'a` ke dalam kolom pencarian dan kirimkan.
   
2. **Verifikasi Eksekusi:**
   - Setelah halaman dihasilkan, fungsi `alert(1)` akan dijalankan secara otomatis, menampilkan kotak dialog alert dengan pesan `1`.

Dengan demikian, meskipun tanda kurung sudut di-HTML encode, kemampuan untuk memecahkan string JavaScript dan menyisipkan perintah tambahan memungkinkan eksekusi kode JavaScript, menyebabkan kerentanan Reflected XSS.
