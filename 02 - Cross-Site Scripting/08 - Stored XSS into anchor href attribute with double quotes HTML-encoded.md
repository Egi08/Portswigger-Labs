
# Stored XSS into anchor href attribute with double quotes HTML-encoded

Lab ini mengandung kerentanan **stored cross-site scripting (XSS)** pada fungsi komentar. Untuk menyelesaikan lab ini, kirimkan komentar yang memanggil fungsi `alert` ketika nama penulis komentar diklik.

---------------------------------------------

Referensi:

- [Contexts dalam Cross-Site Scripting - PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts)

![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/1.png)

---------------------------------------------

Anda dapat memposting komentar:

![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/2.png)

Ini adalah elemen HTML yang dihasilkan:

![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/3.png)

Ketika nama pengguna diklik, akan diarahkan ke situs web yang ditetapkan dalam komentar:

![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/4.png)
![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/5.png)

Kita akan mengatur situs web ke URL JavaScript:

```
javascript:alert(1)
```

![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/6.png)

Ketika diklik, fungsi `alert` akan dijalankan:

![img](images/Stored%20XSS%20into%20anchor%20href%20attribute%20with%20double%20quotes%20HTML-encoded/7.png)

### Penjelasan Query

Pada contoh di atas, kita memanfaatkan kerentanan **Stored XSS** dengan menyuntikkan payload ke dalam atribut `href` pada tag `<a>` yang telah di-HTML-encode untuk tanda kutip ganda. Meskipun tanda kutip ganda (`"`) di-encode, kita masih dapat menyisipkan atribut tambahan atau merusak struktur HTML untuk menjalankan JavaScript.

**Payload yang digunakan adalah:**

```
javascript:alert(1)
```

**Proses Penyisipan dan Eksekusi:**

1. **Menyisipkan Payload:**
   - Saat memposting komentar, masukkan `javascript:alert(1)` sebagai URL situs web dalam komentar.
   - Meskipun tanda kutip ganda di-encode, browser akan menginterpretasikan URL tersebut sebagai skema JavaScript.

2. **Elemen HTML yang Dihasilkan:**
   - Komentar yang dikirim akan menghasilkan elemen HTML seperti berikut:
     ```html
     <a href="javascript:alert(1)">NamaPengguna</a>
     ```
   - Karena atribut `href` mengandung skema `javascript:`, saat link tersebut diklik, browser akan mengeksekusi kode JavaScript yang ada di dalamnya.

3. **Eksekusi JavaScript:**
   - Ketika pengguna mengklik nama penulis komentar, fungsi `alert(1)` akan dijalankan, menampilkan kotak dialog alert dengan pesan `1`.

**Mengapa Payload Berhasil:**

- **Skema `javascript:`:** Browser mendukung skema URI `javascript:`, yang memungkinkan eksekusi kode JavaScript langsung dari atribut `href`.
- **HTML-Encoding Terbatas:** Meskipun tanda kutip ganda di-encode, penyisipan skema `javascript:` tidak terhalang karena tidak memerlukan pemecahan tanda kutip untuk eksekusi.
- **Stored XSS:** Payload disimpan di server dan ditampilkan kembali ke semua pengguna yang melihat komentar, memungkinkan eksekusi JavaScript saat link diklik oleh siapa saja.

**Langkah-langkah Penyelesaian Lab:**

1. **Kirim Komentar:**
   - Isi kolom komentar dengan nama pengguna dan atur URL situs web ke `javascript:alert(1)`.

2. **Verifikasi Eksekusi:**
   - Setelah komentar disimpan, klik pada nama pengguna yang baru saja dikirim.
   - Jika payload berhasil, fungsi `alert(1)` akan dijalankan, menampilkan kotak dialog alert.

Dengan demikian, meskipun terdapat HTML-encode pada tanda kutip ganda, penggunaan skema `javascript:` pada atribut `href` memungkinkan eksekusi kode JavaScript, menyebabkan kerentanan Stored XSS.
