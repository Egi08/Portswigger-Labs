### Cross-Site Scripting (XSS)

Pada bagian ini, kami akan menjelaskan apa itu **Cross-Site Scripting (XSS)**, berbagai jenis kerentanan XSS, serta cara menemukan dan mencegahnya.

#### Apa itu Cross-Site Scripting (XSS)?
**Cross-Site Scripting (XSS)** adalah **kerentanan keamanan pada web** yang memungkinkan **penyerang untuk mengganggu interaksi pengguna** dengan aplikasi yang rentan. XSS memungkinkan penyerang **melewati kebijakan asal** yang memisahkan berbagai situs web. Biasanya, XSS memungkinkan penyerang untuk **berpura-pura menjadi pengguna lain**, melakukan tindakan yang dapat dilakukan oleh pengguna tersebut, dan **mengakses data pengguna**. Jika pengguna yang menjadi korban memiliki **hak akses tinggi**, penyerang bisa **mengendalikan seluruh aplikasi dan datanya**.

#### Bagaimana Cara Kerja XSS?
XSS bekerja dengan **memanipulasi situs web yang rentan** sehingga mengirimkan **JavaScript berbahaya** ke pengguna. Ketika kode berbahaya ini dijalankan di browser korban, penyerang dapat **mengendalikan interaksi korban** dengan aplikasi tersebut.
![image](https://github.com/user-attachments/assets/e6c8d4fc-825d-4de3-b516-c9afc711a2f4)

#### Jenis-Jenis Serangan XSS
Ada **tiga jenis utama serangan XSS**:

1. **Reflected XSS**: Script berbahaya datang dari permintaan HTTP saat itu juga.
2. **Stored XSS**: Script berbahaya disimpan di database situs web dan ditampilkan ke pengguna lain.
3. **DOM-based XSS**: Kerentanan ada pada kode sisi klien (JavaScript) yang memproses data dari sumber yang tidak terpercaya.

##### 1. Reflected XSS
**Reflected XSS** adalah jenis XSS yang paling sederhana. Terjadi ketika aplikasi menerima data dari permintaan HTTP dan menampilkannya kembali **tanpa pemeriksaan yang tepat**.

Contoh sederhana kerentanan Reflected XSS:

```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

Aplikasi tidak melakukan pemrosesan lain terhadap data tersebut, sehingga penyerang dapat dengan mudah membuat serangan seperti ini:

```
https://insecure-website.com/status?message=<script>/* Bad stuff here... */</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

Jika pengguna mengunjungi URL yang dibuat oleh penyerang, maka **script penyerang akan dijalankan** di browser pengguna dalam konteks sesi pengguna dengan aplikasi tersebut. Pada titik itu, script dapat **melakukan tindakan apapun** dan **mengambil data yang dapat diakses oleh pengguna**.

##### 2. Stored XSS
**Stored XSS** (juga dikenal sebagai **persistent atau second-order XSS**) terjadi ketika aplikasi menerima data dari **sumber yang tidak terpercaya** dan **menyimpannya** untuk ditampilkan pada respons HTTP berikutnya **tanpa pemeriksaan yang tepat**.

Data tersebut bisa dikirimkan melalui permintaan HTTP, misalnya **komentar pada posting blog**, **nickname pengguna di ruang obrolan**, atau **detail kontak pada pesanan pelanggan**. Contoh sederhana kerentanan Stored XSS:

Sebuah aplikasi papan pesan memungkinkan pengguna mengirimkan pesan yang ditampilkan kepada pengguna lain:

```
<p>Hello, this is my message!</p>
```

Aplikasi tidak melakukan pemrosesan lain terhadap data tersebut, sehingga penyerang dapat dengan mudah mengirimkan pesan yang menyerang pengguna lain:

```
<p><script>/* Bad stuff here... */</script></p>
```

Pesan ini akan **dijalankan di browser semua pengguna yang melihatnya**.

##### 3. DOM-based XSS
**DOM-based XSS** (juga dikenal sebagai **DOM XSS**) terjadi ketika aplikasi mengandung **JavaScript sisi klien** yang memproses data dari **sumber yang tidak terpercaya** dengan cara yang tidak aman, biasanya dengan **menulis data kembali ke DOM**.

Contoh:
```javascript
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

Jika penyerang dapat **mengontrol nilai dari input `search`**, mereka dapat dengan mudah memasukkan **nilai berbahaya** yang menyebabkan **script mereka dieksekusi**:

```
You searched for: <img src=1 onerror='/* Bad stuff here... */'>
```

Dalam kasus tipikal, input field akan diisi dari bagian permintaan HTTP, seperti **parameter query string URL**, memungkinkan penyerang mengirimkan serangan menggunakan **URL berbahaya**, mirip dengan **Reflected XSS**.

#### Apa yang Bisa Dilakukan dengan XSS?
Penyerang yang berhasil mengeksploitasi kerentanan XSS biasanya dapat:

- **Meniru identitas pengguna lain**.
- **Melakukan tindakan atas nama pengguna**.
- **Membaca data yang dapat diakses pengguna**.
- **Mencuri kredensial login pengguna**.
- **Mengubah tampilan situs web** (**defacement**).
- **Menyuntikkan fungsi berbahaya** ke situs web.

#### Dampak Kerentanan XSS
Dampak serangan XSS tergantung pada aplikasi dan data yang dimilikinya:

- Pada aplikasi sederhana dengan data publik, **dampaknya minimal**.
- Pada aplikasi yang menyimpan data sensitif seperti **transaksi perbankan**, **email**, atau **catatan kesehatan**, **dampaknya serius**.
- Jika pengguna yang dikompromikan memiliki **hak akses tinggi**, penyerang bisa **mengendalikan seluruh aplikasi dan data pengguna lainnya**.

#### Cara Menemukan dan Menguji Kerentanan XSS
Sebagian besar kerentanan XSS dapat ditemukan dengan cepat menggunakan alat seperti **Burp Suite's web vulnerability scanner**.

**Pengujian manual** untuk Reflected dan Stored XSS biasanya melibatkan:

1. **Memasukkan input unik** (seperti string alfanumerik pendek) ke setiap titik masuk aplikasi.
2. **Mengidentifikasi setiap lokasi** di mana input tersebut dikembalikan dalam respons HTTP.
3. **Menguji setiap lokasi** secara individual untuk menentukan apakah input yang dirancang khusus dapat digunakan untuk **mengeksekusi JavaScript arbitrer**.

Dengan cara ini, Anda dapat menentukan **konteks di mana XSS terjadi** dan memilih **payload yang sesuai** untuk mengeksploitasinya.

**Pengujian manual** untuk DOM-based XSS yang berasal dari parameter URL melibatkan proses serupa:

1. **Memasukkan input unik** ke parameter tersebut.
2. Menggunakan **alat pengembang browser** untuk mencari input tersebut di DOM.
3. **Menguji setiap lokasi** untuk menentukan apakah dapat dieksploitasi.

Namun, jenis DOM XSS lainnya **lebih sulit dideteksi**. Untuk menemukan kerentanan DOM-based pada **input yang tidak berbasis URL** (seperti `document.cookie`) atau **sink yang tidak berbasis HTML** (seperti `setTimeout`), tidak ada pengganti selain **meninjau kode JavaScript**, yang bisa sangat memakan waktu. **Burp Suite's web vulnerability scanner** menggabungkan **analisis statis dan dinamis** dari JavaScript untuk secara andal **mengotomatiskan deteksi kerentanan DOM-based**.

#### Kebijakan Keamanan Konten (CSP)
**Content Security Policy (CSP)** adalah mekanisme di browser yang bertujuan **mengurangi dampak XSS** dan beberapa kerentanan lainnya. Jika aplikasi yang menggunakan CSP mengandung perilaku mirip XSS, maka **CSP mungkin dapat menghambat atau mencegah eksploitasi kerentanan** tersebut. Namun, seringkali CSP dapat **dilewati** untuk memungkinkan **eksploitasi kerentanan yang mendasarinya**.

#### Pencegahan Serangan XSS
Mencegah **Cross-Site Scripting** bisa **sederhana** dalam beberapa kasus, tetapi bisa **jauh lebih sulit** tergantung pada **kompleksitas aplikasi** dan cara aplikasi **menangani data yang dapat dikontrol pengguna**.

Secara umum, pencegahan kerentanan XSS yang efektif melibatkan kombinasi langkah-langkah berikut:

1. **Filter Input pada Saat Masuk**: Pada titik di mana input pengguna diterima, **saring secara ketat** berdasarkan apa yang diharapkan atau input yang valid.
2. **Encode Data pada Saat Output**: Pada titik di mana data yang dapat dikontrol pengguna ditampilkan dalam respons HTTP, **encode output** untuk mencegahnya diinterpretasikan sebagai **konten aktif**. Tergantung pada konteks output, ini mungkin memerlukan penerapan kombinasi encoding **HTML**, **URL**, **JavaScript**, dan **CSS**.
3. **Gunakan Header Respons yang Tepat**: Untuk mencegah XSS pada respons HTTP yang tidak dimaksudkan untuk mengandung HTML atau JavaScript, Anda dapat menggunakan header **`Content-Type`** dan **`X-Content-Type-Options`** untuk memastikan bahwa browser **menginterpretasikan respons sesuai dengan yang dimaksudkan**.
4. **Content Security Policy (CSP)**: Sebagai **garis pertahanan terakhir**, Anda dapat menggunakan CSP untuk **mengurangi tingkat keparahan kerentanan XSS** yang masih terjadi.

#### Pertanyaan Umum tentang XSS

- **Seberapa umum kerentanan XSS?**
  XSS sangat **umum** dan mungkin merupakan **kerentanan keamanan web yang paling sering terjadi**.

- **Seberapa umum serangan XSS?**
  Sulit untuk mendapatkan data yang dapat diandalkan tentang serangan XSS di dunia nyata, tetapi kemungkinan **dieksploitasi kurang sering** dibandingkan kerentanan lainnya.

- **Apa perbedaan antara XSS dan CSRF?**
  XSS melibatkan **penyisipan JavaScript berbahaya** ke situs web, sementara CSRF membuat pengguna **melakukan tindakan yang tidak mereka inginkan**.

- **Apa perbedaan antara XSS dan SQL Injection?**
  XSS adalah **kerentanan sisi klien** yang menargetkan pengguna lain, sementara **SQL Injection adalah kerentanan sisi server** yang menargetkan database aplikasi.

- **Bagaimana cara mencegah XSS di PHP?**
  Saring input dengan **daftar karakter yang diizinkan** dan gunakan **type hints atau type casting**. **Escape output** dengan `htmlentities` dan `ENT_QUOTES` untuk konteks HTML, atau menggunakan **JavaScript Unicode escapes** untuk konteks JavaScript.

- **Bagaimana cara mencegah XSS di Java?**
  Saring input dengan **daftar karakter yang diizinkan** dan gunakan **library seperti Google Guava** untuk **HTML-encode output** untuk konteks HTML, atau gunakan **JavaScript Unicode escapes** untuk konteks JavaScript.

### Praktik Lab XSS
Jika Anda sudah memahami **konsep dasar XSS** dan ingin **berlatih**, Anda bisa mengakses berbagai **lab** yang dirancang khusus untuk **mengeksploitasi XSS dengan aman**.

**[Lihat Semua Lab XSS](#)**

Dengan memahami dan menerapkan langkah-langkah di atas, Anda dapat **melindungi aplikasi web Anda** dari serangan **Cross-Site Scripting**.
