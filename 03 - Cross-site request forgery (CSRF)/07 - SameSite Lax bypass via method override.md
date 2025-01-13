# SameSite Lax Bypass melalui Method Override

Fungsi perubahan email pada lab ini rentan terhadap serangan CSRF. Untuk menyelesaikan lab ini, lakukan serangan CSRF yang mengubah alamat email korban. Anda harus menggunakan server exploit yang disediakan untuk meng-host serangan Anda.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: `wiener:peter`

**Catatan:** Pembatasan SameSite default berbeda antara browser. Karena korban menggunakan Chrome, kami menyarankan untuk juga menggunakan Chrome (atau browser Chromium bawaan Burp) untuk menguji exploit Anda.

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji exploit Anda, pastikan untuk menggunakan alamat email yang berbeda untuk exploit akhir yang akan Anda kirimkan kepada korban.

---------------------------------------------

**Referensi:**

- [Bypassing SameSite Restrictions](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions)

![img](images/SameSite%20Lax%20bypass%20via%20method%20override/1.png)

---------------------------------------------

Chrome secara default menerapkan pembatasan SameSite Lax, sehingga kita harus mencoba mengeksekusi permintaan GET yang akan mengubah email pengguna. Secara default, permintaan ini adalah POST:

![img](images/SameSite%20Lax%20bypass%20via%20method%20override/2.png)

Jika kita mengubah metode menjadi GET dan menambahkan parameter “_method”, serangan tetap berhasil:

![img](images/SameSite%20Lax%20bypass%20via%20method%20override/3.png)

Kemudian, kita akan menggunakan payload berikut:

```html
<script>
    document.location = 'https://0a6d00f7038377ec8069495800fd0010.web-security-academy.net/my-account/change-email?email=test4%40test.com&_method=POST';
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

Pada kasus ini, Chrome menggunakan pembatasan **SameSite Lax** secara default, yang berarti cookie akan dikirim untuk permintaan GET lintas situs, tetapi tidak untuk permintaan POST.

### Bagaimana Method Override Dapat Mengakali SameSite Lax?

Meskipun SameSite Lax membatasi pengiriman cookie untuk permintaan POST lintas situs, teknik **method override** memungkinkan penyerang untuk mengubah metode permintaan. Dengan menambahkan parameter `_method=POST` dalam permintaan GET, server dapat memproses permintaan tersebut seolah-olah itu adalah permintaan POST.

### Langkah-Langkah Serangan

1. **Menyiapkan Halaman Exploit:** Penyerang membuat halaman HTML yang berisi skrip yang mengarahkan browser korban untuk mengirim permintaan GET yang disamarkan sebagai POST menggunakan parameter `_method=POST`.

2. **Menggunakan Permintaan GET dengan Parameter `_method`:** Payload mengubah metode permintaan dari GET ke POST dengan menyertakan parameter `_method=POST`. Karena permintaan awal adalah GET, cookie SameSite Lax akan disertakan.

3. **Mengirimkan Permintaan:** Ketika korban mengunjungi halaman exploit, skrip secara otomatis mengarahkan browser untuk mengirimkan permintaan yang mengubah alamat email mereka.

4. **Mengubah Email Korban:** Karena metode permintaan telah diubah menjadi POST dan cookie CSRF disertakan, aplikasi web memproses permintaan tersebut sebagai permintaan yang sah untuk mengubah email korban.

### Mengapa Kerentanan Ini Terjadi?

Kerentanan ini terjadi karena:

- **Implementasi Method Override yang Tidak Aman:** Server mengizinkan pengubahan metode permintaan melalui parameter `_method` tanpa memvalidasi asal permintaan tersebut. Hal ini memungkinkan penyerang untuk mengubah metode permintaan dari GET ke POST.

- **SameSite Lax Terlewati oleh Permintaan GET:** Karena SameSite Lax mengizinkan pengiriman cookie pada permintaan GET lintas situs, penyerang dapat memanfaatkan ini untuk menyertakan cookie CSRF dalam permintaan yang diubah menjadi POST.

- **Kurangnya Validasi CSRF yang Kuat:** Aplikasi web tidak memiliki mekanisme validasi CSRF yang cukup kuat untuk memastikan bahwa permintaan tersebut berasal dari sumber yang sah, sehingga memungkinkan serangan ini berhasil.

## Kesimpulan

Untuk mencegah kerentanan CSRF seperti ini, penting untuk:

- **Menggunakan Atribut SameSite yang Tepat:** Pertimbangkan untuk menggunakan `SameSite=Strict` atau `SameSite=Lax` dengan hati-hati, dan pastikan bahwa pengaturan ini sesuai dengan kebutuhan aplikasi.

- **Menghindari Method Override yang Tidak Aman:** Jika menggunakan teknik method override, pastikan bahwa permintaan yang diubah melalui parameter tambahan divalidasi dengan ketat untuk mencegah penyalahgunaan.

- **Menerapkan Validasi CSRF yang Lebih Kuat:** Selain menggunakan token CSRF, pastikan bahwa token tersebut dihasilkan secara acak, unik untuk setiap sesi, dan diverifikasi dengan benar pada setiap permintaan yang sensitif.

- **Menggunakan Atribut Secure dan HttpOnly pada Cookie:** Ini membantu mencegah akses tidak sah ke cookie dari sisi klien dan mengurangi risiko serangan XSS yang dapat mencuri token CSRF.

Dengan menerapkan langkah-langkah ini, aplikasi web dapat lebih efektif dalam melindungi diri dari serangan CSRF dan memastikan bahwa tindakan sensitif hanya dapat dilakukan oleh pengguna yang sah.
