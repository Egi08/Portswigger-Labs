### Cross-Site Scripting (XSS)

Pada bagian ini, kita akan membahas apa itu **Cross-Site Scripting (XSS)**, jenis-jenisnya, serta cara menemukannya dan mencegahnya.

#### Apa itu Cross-Site Scripting (XSS)?
**Cross-Site Scripting (XSS)** adalah **kerentanan keamanan pada website** yang memungkinkan **penyerang mengganggu interaksi pengguna** dengan aplikasi yang rentan. Dengan XSS, penyerang bisa **melewati batasan keamanan** yang memisahkan berbagai situs web. Biasanya, XSS memungkinkan penyerang untuk **berpura-pura menjadi pengguna lain**, melakukan tindakan yang bisa dilakukan oleh pengguna tersebut, dan **mengakses data pengguna**. Jika korban memiliki **hak akses tinggi**, penyerang bisa **mengontrol seluruh aplikasi dan datanya**.

#### Bagaimana Cara Kerja XSS?
XSS bekerja dengan **mengubah website yang rentan** sehingga mengirimkan **kode JavaScript berbahaya** ke pengguna. Ketika kode ini dijalankan di browser korban, penyerang bisa **mengendalikan interaksi korban** dengan aplikasi tersebut.

![image](https://github.com/user-attachments/assets/e6c8d4fc-825d-4de3-b516-c9afc711a2f4)


#### Jenis-Jenis Serangan XSS
Ada **tiga jenis utama XSS**:

1. **Reflected XSS**: Script berbahaya datang dari permintaan HTTP langsung.
2. **Stored XSS**: Script berbahaya disimpan di database website dan ditampilkan ke pengguna lain.
3. **DOM-based XSS**: Kerentanan ada pada kode JavaScript di sisi pengguna yang memproses data dari sumber yang tidak terpercaya.

##### 1. Reflected XSS
**Reflected XSS** adalah jenis XSS yang paling sederhana. Terjadi ketika aplikasi menerima data dari permintaan HTTP dan langsung menampilkannya kembali **tanpa pemeriksaan**.

**Contoh Reflected XSS:**
```
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

Jika penyerang mengirimkan URL dengan script berbahaya:
```
https://insecure-website.com/status?message=<script>/* Bad stuff here... */</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

Script ini akan dijalankan di browser korban saat mereka mengunjungi URL tersebut, memungkinkan penyerang **melakukan tindakan apapun** dan **mengambil data yang bisa diakses** oleh pengguna.

##### 2. Stored XSS
**Stored XSS** terjadi ketika data berbahaya disimpan di server dan ditampilkan ke pengguna lain tanpa pemeriksaan yang tepat.

**Contoh Stored XSS:**
Sebuah aplikasi papan pesan memungkinkan pengguna mengirimkan pesan:
```
<p>Hello, this is my message!</p>
```

Jika penyerang mengirim pesan berbahaya:
```
<p><script>/* Bad stuff here... */</script></p>
```

Pesan ini akan **dijalankan di browser semua pengguna** yang melihatnya.

##### 3. DOM-based XSS
**DOM-based XSS** terjadi ketika JavaScript di sisi pengguna memproses data dari sumber yang tidak terpercaya tanpa aman.

**Contoh DOM-based XSS:**
```javascript
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

Jika penyerang mengontrol nilai `search`, mereka bisa memasukkan kode berbahaya:
```
You searched for: <img src=1 onerror='/* Bad stuff here... */'>
```

#### Apa yang Bisa Dilakukan dengan XSS?
Penyerang yang berhasil menggunakan XSS bisa:
- **Meniru identitas pengguna lain**.
- **Melakukan tindakan atas nama pengguna**.
- **Membaca data pengguna**.
- **Mencuri informasi login**.
- **Mengubah tampilan situs web**.
- **Menyuntikkan fungsi berbahaya** ke situs web.

#### Dampak Kerentanan XSS
Dampak serangan XSS tergantung pada aplikasi dan data yang dimilikinya:
- **Aplikasi sederhana** dengan data publik: **dampak minimal**.
- **Aplikasi dengan data sensitif** seperti transaksi perbankan atau catatan kesehatan: **dampak serius**.
- Jika **pengguna dengan hak akses tinggi** terpengaruh: penyerang bisa **mengendalikan seluruh aplikasi**.

#### Cara Menemukan dan Menguji Kerentanan XSS
Sebagian besar kerentanan XSS bisa ditemukan dengan alat seperti **Burp Suite's web vulnerability scanner**.

**Pengujian Manual:**
1. **Masukkan input unik** (seperti kata atau angka tertentu) di setiap titik masuk aplikasi.
2. **Cari lokasi** di mana input tersebut muncul di respons HTTP.
3. **Uji setiap lokasi** untuk melihat apakah input bisa menjalankan JavaScript.

**Pengujian DOM-based XSS:**
1. **Masukkan input unik** ke parameter URL.
2. Gunakan **alat pengembang browser** untuk mencari input di DOM.
3. **Uji setiap lokasi** untuk melihat apakah bisa dieksploitasi.

#### Kebijakan Keamanan Konten (CSP)
**Content Security Policy (CSP)** adalah mekanisme di browser yang membantu **mengurangi dampak XSS** dengan membatasi sumber konten yang diizinkan. Namun, CSP kadang bisa **dilewati** oleh penyerang.

#### Pencegahan Serangan XSS
Untuk mencegah XSS, lakukan langkah-langkah berikut:

1. **Filter Input saat Masuk**:
   - Saring input pengguna sesuai dengan yang diharapkan.
   
2. **Encode Output saat Ditampilkan**:
   - Ubah data sebelum ditampilkan agar tidak bisa dijalankan sebagai kode.
   
3. **Gunakan Header Respons yang Tepat**:
   - Pastikan respons HTTP memiliki header seperti `Content-Type` untuk menghindari interpretasi yang salah oleh browser.
   
4. **Gunakan CSP**:
   - Tambahkan CSP sebagai lapisan perlindungan tambahan.

#### Pertanyaan Umum tentang XSS

- **Seberapa umum kerentanan XSS?**
  XSS sangat **umum** dan mungkin merupakan kerentanan keamanan web yang paling sering terjadi.

- **Seberapa sering serangan XSS terjadi?**
  Sulit untuk mendapatkan data pasti, tetapi kemungkinan **kurang sering** dibandingkan kerentanan lainnya.

- **Apa perbedaan antara XSS dan CSRF?**
  - **XSS**: Menyisipkan JavaScript berbahaya ke situs web.
  - **CSRF**: Membuat pengguna melakukan tindakan yang tidak mereka inginkan.

- **Apa perbedaan antara XSS dan SQL Injection?**
  - **XSS**: Kerentanan di sisi pengguna yang menargetkan pengguna lain.
  - **SQL Injection**: Kerentanan di sisi server yang menargetkan database aplikasi.

- **Bagaimana cara mencegah XSS di PHP?**
  - Saring input dengan daftar karakter yang diizinkan.
  - Gunakan `htmlentities` untuk mengubah data sebelum ditampilkan.
  - Hindari menyisipkan data pengguna langsung ke HTML atau JavaScript.

- **Bagaimana cara mencegah XSS di Java?**
  - Saring input dengan daftar karakter yang diizinkan.
  - Gunakan library seperti **Google Guava** untuk mengubah data sebelum ditampilkan.
  - Hindari menyisipkan data pengguna langsung ke HTML atau JavaScript.

### Latihan Lab XSS
Jika Anda sudah memahami **dasar-dasar XSS** dan ingin **berlatih**, Anda bisa mengakses berbagai **lab** yang dibuat khusus untuk **menguji XSS dengan aman**.

**[Lihat Semua Lab XSS](#)**

Dengan memahami dan menerapkan langkah-langkah di atas, Anda dapat **melindungi aplikasi web Anda** dari serangan **Cross-Site Scripting**.
