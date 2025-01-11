
# Stored DOM XSS
### Proof of Concept (PoC) untuk Stored DOM XSS

#### Deskripsi Kerentanan

Lab ini menunjukkan **kerentanan Cross-Site Scripting (XSS) berbasis DOM yang tersimpan (Stored DOM XSS)** pada fungsi komentar blog. Kerentanan ini memungkinkan penyerang untuk menyimpan skrip berbahaya dalam komentar, yang kemudian dijalankan di browser pengguna lain saat mereka melihat komentar tersebut.

Untuk menyelesaikan lab ini, Anda perlu mengeksploitasi kerentanan ini untuk memanggil fungsi `alert()`.

---------------------------------------------

#### Referensi:

- [PortSwigger: DOM-Based Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/dom-based)

---------------------------------------------

#### Langkah-Langkah Proof of Concept (PoC)

##### 1. Memahami Fungsi Komentar yang Rentan

Aplikasi ini memiliki fitur untuk memposting komentar pada blog. Namun, fungsi ini rentan terhadap serangan Stored DOM XSS karena tidak memproses input pengguna dengan benar sebelum menyimpannya dan menampilkannya kembali di halaman blog.

**Fungsi Memposting Komentar:**

![Fungsi Memposting Komentar](images/Stored%20DOM%20XSS/1.png)

##### 2. Analisis HTML yang Dihasilkan

Saat pengguna memposting komentar, aplikasi menghasilkan kode HTML berikut untuk menampilkan komentar tersebut:

![HTML Komentar](images/Stored%20DOM%20XSS/2.png)

Perhatikan bahwa input komentar pengguna dimasukkan langsung ke dalam elemen `<p>` tanpa sanitasi yang memadai.

##### 3. Membuat dan Menyisipkan Payload XSS

Untuk mengeksploitasi kerentanan ini, kita dapat menyisipkan payload berbahaya ke dalam komentar yang akan dijalankan di browser pengguna lain. Berikut adalah contoh payload yang dapat digunakan:

```
</p><img src=x onerror=alert(1) /><p>
```

**Penjelasan Payload:**

- `</p>`: Menutup tag `<p>` yang ada.
- `<img src=x onerror=alert(1) />`: Menyisipkan elemen `<img>` dengan atribut `src` yang tidak valid (`x`), sehingga menyebabkan error dan menjalankan `onerror`, yang memanggil fungsi `alert(1)`.
- `<p>`: Membuka kembali tag `<p>` agar struktur HTML tetap valid.

##### 4. Menyisipkan Payload ke dalam Komentar

Masukkan payload berikut ke dalam kotak komentar dan kirimkan:

```
</p><img src=x onerror=alert(1) /><p>
```

![Payload XSS](images/Stored%20DOM%20XSS/3.png)

##### 5. Mengamati Eksekusi Kode Berbahaya

Setelah komentar dengan payload disimpan dan ditampilkan kembali di halaman blog, browser korban akan menjalankan skrip berbahaya tersebut. Sebuah popup alert akan muncul, menunjukkan bahwa XSS berhasil dieksekusi.

![Eksekusi Payload](images/Stored%20DOM%20XSS/4.png)

#### Penjelasan Query dan Payload

- **Proses Penyisipan Komentar:**

  Saat pengguna memasukkan komentar, data tersebut dikirimkan ke server melalui permintaan HTTP (biasanya metode POST). Server kemudian menyimpan komentar tersebut dalam basis data dan mengirimkannya kembali dalam respons HTTP saat halaman blog dimuat ulang.

- **Bagian Rentan dalam Kode:**

  Pada kode JavaScript yang mengelola tampilan komentar, input pengguna dimasukkan langsung ke dalam elemen `<p>` tanpa sanitasi atau encoding yang tepat. Hal ini memungkinkan penyerang untuk menyisipkan elemen HTML dan atribut JavaScript berbahaya.

- **Payload XSS:**

  ```
  </p><img src=x onerror=alert(1) /><p>
  ```

  - **`</p>`**: Menutup elemen `<p>` yang ada untuk memastikan bahwa elemen selanjutnya (gambar) berada di luar konteks paragraf.
  - **`<img src=x onerror=alert(1) />`**: Menyisipkan gambar dengan sumber yang tidak valid (`x`), yang menyebabkan error loading gambar dan memicu event `onerror`, menjalankan fungsi `alert(1)`.
  - **`<p>`**: Membuka kembali elemen `<p>` untuk menjaga struktur HTML tetap rapi dan menghindari kerusakan tata letak.

#### Langkah-Langkah Eksploitasi

1. **Memasukkan Payload ke dalam Komentar:**

   Pada kotak komentar, masukkan payload berikut dan kirimkan:

   ```
   </p><img src=x onerror=alert(1) /><p>
   ```

2. **Mengirimkan Permintaan Komentar:**

   Setelah mengirimkan komentar, server akan menyimpan komentar tersebut dan mengembalikannya dalam respons HTTP saat halaman blog dimuat ulang.

3. **Amati Eksekusi Kode:**

   Saat halaman blog ditampilkan dengan komentar yang mengandung payload, browser akan mengeksekusi skrip berbahaya tersebut. Sebuah popup alert dengan pesan "1" akan muncul di browser korban, menandakan bahwa XSS berhasil dieksekusi.

   ```
   <p></p><img src=x onerror=alert(1) /><p> search results for '</p><img src=x onerror=alert(1) /><p>'
   ```

   Popup alert akan muncul seperti ini:

   ![Eksekusi Payload](images/Stored%20DOM%20XSS/4.png)

#### Pencegahan Kerentanan Stored DOM XSS

Untuk mencegah kerentanan Stored DOM XSS seperti ini dalam aplikasi Anda, berikut beberapa langkah yang dapat diambil:

1. **Sanitasi dan Validasi Input Pengguna:**
   - Selalu bersihkan dan validasi input pengguna sebelum menyimpannya atau menampilkannya kembali di halaman web.
   - Gunakan pustaka sanitasi yang terpercaya untuk menghapus atau mengkodekan karakter berbahaya.

2. **Gunakan Fungsi `textContent` atau `innerText` Daripada `innerHTML`:**
   - Jika memungkinkan, gunakan `textContent` atau `innerText` untuk menambahkan teks ke elemen DOM daripada `innerHTML`, yang bisa dieksploitasi untuk menyisipkan HTML atau JavaScript berbahaya.

3. **Hindari Penggunaan `eval()`:**
   - Fungsi `eval()` dapat menjalankan kode JavaScript dinamis dan harus dihindari kecuali benar-benar diperlukan. Sebagai gantinya, gunakan metode parsing JSON yang aman seperti `JSON.parse()`.

4. **Implementasikan Content Security Policy (CSP):**
   - CSP dapat membantu mengurangi dampak serangan XSS dengan membatasi sumber konten yang diizinkan untuk dijalankan di browser.

5. **Gunakan Framework yang Aman:**
   - Banyak framework modern memiliki mekanisme built-in untuk mencegah XSS. Pastikan untuk memanfaatkan fitur-fitur ini dan tetap memperbarui framework Anda ke versi terbaru.

6. **Audit dan Uji Keamanan Secara Berkala:**
   - Lakukan audit kode dan pengujian penetrasi secara rutin untuk mengidentifikasi dan memperbaiki kerentanan sebelum dapat dimanfaatkan oleh penyerang.

Dengan menerapkan langkah-langkah di atas, Anda dapat secara signifikan mengurangi risiko serangan Stored DOM XSS dan meningkatkan keamanan aplikasi web Anda.

### Kesimpulan

Stored DOM XSS adalah salah satu jenis kerentanan yang sangat berbahaya karena memungkinkan penyerang untuk menyisipkan dan menjalankan skrip berbahaya yang tersimpan di server, yang kemudian dijalankan di browser pengguna lain. Dengan memahami bagaimana kerentanan ini bekerja dan bagaimana cara mengeksploitasinya, pengembang dapat lebih waspada dan menerapkan langkah-langkah pencegahan yang efektif untuk melindungi aplikasi web dan data pengguna.

Selalu prioritaskan keamanan dalam pengembangan aplikasi untuk melindungi data pengguna dan menjaga integritas sistem Anda.
