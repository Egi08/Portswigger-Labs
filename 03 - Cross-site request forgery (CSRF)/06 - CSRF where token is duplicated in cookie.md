# CSRF di Mana Token Diduplicasi dalam Cookie

Fungsi perubahan email pada lab ini rentan terhadap serangan CSRF. Hal ini terjadi karena mencoba menggunakan teknik pencegahan CSRF yang tidak aman yaitu "double submit".

Untuk menyelesaikan lab ini, gunakan server exploit Anda untuk meng-host halaman HTML yang memanfaatkan serangan CSRF untuk mengubah alamat email pengguna yang sedang melihat halaman tersebut.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: `wiener:peter`

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji exploit Anda, pastikan untuk menggunakan alamat email yang berbeda untuk exploit akhir yang akan Anda kirimkan kepada korban.

---------------------------------------------

**Referensi:**

- [Bypassing CSRF Token Validation](https://portswigger.net/web-security/csrf/bypassing-token-validation)

![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/1.png)

---------------------------------------------

Kami menemukan nilai token CSRF dalam parameter POST dan header Cookie HTTP, dengan nilai yang sama:

![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/2.png)

Menggunakan nilai acak, serangan masih berhasil:

![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/3.png)

Fungsi pencarian kembali menggunakan parameter `LastSearchTerm` dengan istilah pencarian terakhir:

![img](images/CSRF%20where%20token%20is%20duplicated%20in%20cookie/4.png)

Kami akan memperbarui PoC dari lab sebelumnya:

```html
<form action="https://0ab500c8032ce2a580584e1000e20095.web-security-academy.net/my-account/change-email" method="POST">
     <input type="hidden" name="email" value="tes1t1&#64;gmail&#46;com" />
     <input type="hidden" name="csrf" value="test" />
     <input type="submit" value="Submit request" />
</form>
<img src="https://0ab500c8032ce2a580584e1000e20095.web-security-academy.net/?search=abc%0d%0aSet-Cookie:%20csrf=test%3b%20SameSite=None" onerror="document.forms[0].submit();"/>
```

## Penjelasan

### Apa Itu CSRF?

Cross-Site Request Forgery (CSRF) adalah serangan yang memaksa pengguna akhir untuk menjalankan tindakan yang tidak diinginkan di aplikasi web di mana mereka sedang terautentikasi. Serangan ini mengeksploitasi kepercayaan yang dimiliki situs terhadap browser pengguna.

### Bagaimana CSRF Token Diduplicasi dalam Cookie Menyebabkan Kerentanan?

Dalam kasus ini, aplikasi web menggunakan teknik "double submit" untuk mencegah serangan CSRF. Teknik ini melibatkan pengiriman token CSRF dua kali: sekali dalam parameter POST dan sekali lagi dalam cookie. Namun, jika token tersebut diduplicasi dalam cookie dengan cara yang tidak aman, maka serangan CSRF masih mungkin dilakukan.

#### Mengapa Teknik "Double Submit" Tidak Aman di Sini?

- **Penggunaan Nilai yang Sama:** Jika nilai token CSRF yang dikirim melalui parameter POST sama dengan yang ada di cookie, penyerang dapat memanfaatkan ini dengan menyisipkan token yang valid ke dalam cookie melalui serangan XSS atau metode lainnya.
  
- **Manipulasi Cookie:** Dalam contoh di atas, penyerang dapat menggunakan gambar dengan URL yang diubah untuk menyetel cookie `csrf` ke nilai yang dikontrol (`test`). Karena aplikasi tidak memverifikasi bahwa token yang diterima berasal dari sumber yang sah, serangan ini berhasil.

### Langkah-Langkah Serangan CSRF

1. **Menyiapkan Halaman Exploit:** Penyerang membuat halaman HTML yang berisi form tersembunyi yang mengirimkan permintaan POST untuk mengubah email pengguna. Form ini mencakup token CSRF yang dikontrol (`test`).

2. **Menetapkan Cookie CSRF:** Dengan menggunakan elemen gambar (`img`) yang mencoba memuat URL dengan parameter yang memaksa set-cookie untuk `csrf=test`, penyerang dapat menyetel cookie CSRF pada browser korban.

3. **Mengirimkan Permintaan:** Ketika gambar gagal dimuat, atribut `onerror` akan secara otomatis mengirimkan permintaan POST dengan form yang telah diisi, termasuk token CSRF yang telah diset ke `test`.

4. **Mengubah Email Korban:** Karena aplikasi tidak memverifikasi asal token dengan benar, permintaan POST yang dihasilkan berhasil mengubah alamat email korban.

### Mengapa Kerentanan Ini Terjadi?

Kerentanan ini terjadi karena:

- **Validasi Token yang Lemah:** Aplikasi tidak memverifikasi bahwa token CSRF yang diterima berasal dari sumber yang sah atau unik untuk sesi pengguna.
  
- **Implementasi "Double Submit" yang Tidak Aman:** Penggunaan metode "double submit" tanpa mekanisme tambahan seperti SameSite cookie atau memastikan token dihasilkan secara acak dan unik untuk setiap sesi membuat aplikasi rentan terhadap serangan ini.

- **Kemampuan untuk Menetapkan Cookie Secara Eksternal:** Penyerang dapat memanipulasi cookie CSRF melalui serangan XSS atau metode lain, sehingga memungkinkan mereka untuk mengontrol nilai token yang dikirimkan dalam permintaan.

## Kesimpulan

Untuk mencegah kerentanan CSRF seperti ini, penting untuk menggunakan teknik pencegahan yang lebih aman, seperti penggunaan token CSRF yang dihasilkan secara acak dan unik untuk setiap sesi, serta memastikan bahwa token tersebut hanya dapat diakses melalui header yang aman atau mekanisme lain yang tidak dapat dimanipulasi oleh penyerang. Selain itu, penggunaan atribut `SameSite` pada cookie dapat membantu membatasi pengiriman cookie hanya pada konteks yang sah.
