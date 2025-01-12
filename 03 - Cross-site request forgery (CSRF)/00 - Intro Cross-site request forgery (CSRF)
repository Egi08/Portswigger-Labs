## Cross-site request forgery (CSRF)

**Dalam bagian ini, kami akan menjelaskan apa itu cross-site request forgery**, mendeskripsikan beberapa contoh kerentanan CSRF yang umum, serta **menjelaskan cara mencegah serangan CSRF**.

---

### Apa itu CSRF?
**Cross-site request forgery (juga dikenal sebagai CSRF)** adalah **kerentanan keamanan web** yang memungkinkan penyerang memicu pengguna untuk melakukan aksi yang sebenarnya tidak ingin mereka lakukan. Kerentanan ini memungkinkan penyerang untuk **sebagian menembus perlindungan same-origin policy**, yang pada dasarnya dirancang untuk mencegah situs web berbeda saling mengganggu satu sama lain.

---

### CSRF Labs
Jika Anda sudah memahami konsep dasar di balik kerentanan CSRF dan hanya ingin berlatih mengeksploitasi kerentanan ini pada target yang realistis dan sengaja dibuat rentan, Anda dapat mengakses semua lab pada topik ini melalui tautan di bawah:

**View all CSRF labs**

---

### Apa dampak serangan CSRF?
Dalam serangan CSRF yang berhasil, penyerang menyuruh pengguna (korban) untuk **melakukan aksi** yang sebenarnya tidak mereka niatkan. Contohnya, mengganti alamat email akun mereka, mengubah password, atau mentransfer dana. Tergantung pada jenis aksi tersebut, penyerang **mungkin dapat memperoleh kendali penuh** atas akun pengguna. Jika pengguna yang diserang memiliki **peran (role) istimewa** dalam aplikasi, maka penyerang mungkin bisa **menguasai seluruh data dan fungsi aplikasi**.

---

### Bagaimana CSRF bekerja?
Agar serangan CSRF dapat dilakukan, tiga kondisi utama harus terpenuhi:

1. **Aksi yang relevan**  
   Ada aksi dalam aplikasi yang memiliki **nilai** bagi penyerang untuk dijalankan. Misalnya, aksi berhak istimewa (seperti memodifikasi izin untuk pengguna lain) atau aksi apa pun yang berkaitan dengan data spesifik milik pengguna (misalnya mengubah password pengguna sendiri).

2. **Penanganan sesi berbasis cookie**  
   Proses menjalankan aksi melibatkan satu atau lebih request HTTP, dan aplikasi **hanya mengandalkan session cookie** untuk mengidentifikasi pengguna yang mengajukan request. **Tidak ada mekanisme lain** yang digunakan untuk melacak sesi atau memvalidasi request pengguna.

3. **Tidak ada parameter request yang tidak terduga**  
   Request yang menjalankan aksi tersebut **tidak mengandung parameter** yang nilainya **tidak dapat ditentukan** atau **tidak dapat ditebak** oleh penyerang. Misalnya, jika fungsi mengubah password membutuhkan password lama, maka penyerang perlu tahu password lama tersebut agar dapat memanfaatkan kerentanan.

#### Contoh
Misalkan ada fungsi dalam aplikasi yang memungkinkan pengguna mengganti alamat email akun. Saat pengguna melakukan aksi ini, mereka membuat **request HTTP** seperti berikut:

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

Hal ini memenuhi kondisi untuk CSRF:

1. **Mengubah alamat email** pada akun pengguna adalah aksi yang menarik bagi penyerang. Setelah email diubah, penyerang umumnya bisa **memicu reset password** dan mengambil alih akun korban.
2. Aplikasi hanya mengandalkan **session cookie** untuk mengidentifikasi pengguna yang mengirimkan request. **Tidak ada token atau mekanisme** lain untuk melacak sesi pengguna.
3. **Penyerang dapat dengan mudah menebak** nilai parameter yang dibutuhkan (mis. `email`).

Dengan kondisi tersebut, penyerang dapat **membangun sebuah halaman web** berisi HTML:

```html
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

**Jika** korban membuka halaman penyerang **dan** korban sedang dalam keadaan login di situs target:

1. Halaman penyerang akan memicu **request HTTP** ke situs yang rentan.
2. Karena korban masih login, **browser akan otomatis menyertakan session cookie** ke request tersebut (mengasumsikan cookie SameSite tidak digunakan).
3. Situs yang rentan akan **memproses request** seperti biasa, dan menganggap request itu datang dari korban, sehingga **email korban akan berubah**.

> **Catatan**  
> Meskipun CSRF biasanya dijelaskan dalam konteks **session cookie berbasis HTTP**, kerentanan ini juga dapat terjadi pada konteks lain yang mana aplikasi menambahkan kredensial pengguna ke dalam request secara otomatis, seperti **HTTP Basic authentication** dan **certificate-based authentication**.

---

### Cara membangun serangan CSRF
Membuat HTML **secara manual** untuk eksploitasi CSRF kadang merepotkan, terutama jika request yang dibutuhkan **mengandung banyak parameter** atau memiliki “keanehan” tertentu. **Cara paling mudah** untuk membangun **proof-of-concept (PoC)** CSRF adalah dengan menggunakan **CSRF PoC generator** bawaan di **Burp Suite Professional**:

1. Pilih request apa pun di Burp Suite Professional yang ingin Anda tes atau eksploitasi.
2. Dari menu klik-kanan, pilih **Engagement tools / Generate CSRF PoC**.
3. Burp Suite akan menghasilkan HTML yang **memicu request** terpilih (dikurangi cookie, karena cookie akan otomatis ditambahkan oleh browser korban).
4. Anda dapat menyesuaikan berbagai opsi di generator PoC untuk memaksimalkan serangan. Terkadang ini dibutuhkan untuk menangani situasi request yang unik.
5. Salin HTML yang dihasilkan ke sebuah halaman web, buka di browser yang login ke situs target, dan uji apakah request tersebut berhasil dikirim dan aksi yang diinginkan terjadi.

---

### LAB: APPRENTICE – CSRF vulnerability with no defenses (Solved)
Lab ini adalah contoh di mana aplikasi **benar-benar tidak memiliki perlindungan** terhadap CSRF.

---

### Cara mengirimkan eksploit CSRF
Mekanisme penyebaran (delivery) untuk serangan **cross-site request forgery** pada dasarnya sama dengan serangan **reflected XSS**. Biasanya, penyerang akan menempatkan **HTML berbahaya** pada **situs yang mereka kontrol**, lalu memancing korban untuk mengunjungi situs tersebut. Ini dapat dilakukan dengan **memberikan tautan** ke korban melalui **email** atau **media sosial**. Jika serangan ditempatkan pada situs yang populer (mis. di kolom komentar), penyerang cukup menunggu **pengguna** mengunjungi situs tersebut.

Perlu diperhatikan bahwa beberapa serangan CSRF sederhana **menggunakan method GET** dan bisa berbentuk **satu URL** saja pada situs yang rentan. Jika situasinya seperti ini, penyerang mungkin **tidak perlu** menggunakan situs eksternal, dan dapat langsung menyebarkan **URL jahat** di domain yang rentan. Misalnya, jika request untuk mengubah email bisa dilakukan dengan GET, maka serangan **sepenuhnya mandiri** dapat terlihat seperti ini:

```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

---

### Baca lebih lanjut
- **XSS vs CSRF**
- **Common defences against CSRF**

---

### Pertahanan umum terhadap CSRF
Saat ini, untuk menemukan dan mengeksploitasi kerentanan CSRF, **seringkali penyerang harus mengecoh** langkah-langkah pertahanan CSRF yang diterapkan oleh situs target maupun browser korban. Beberapa pertahanan yang paling umum adalah:

1. **CSRF tokens**  
   Sebuah CSRF token adalah **nilai unik, rahasia, dan tidak terduga** yang dihasilkan oleh aplikasi di sisi server dan dibagikan ke klien. Saat mencoba melakukan aksi sensitif, seperti submit form, klien harus menyertakan CSRF token yang benar di request. Ini menyulitkan penyerang untuk membuat request yang valid atas nama korban.

2. **SameSite cookies**  
   SameSite adalah mekanisme keamanan di browser yang menentukan **kapan sebuah cookie akan disertakan** saat request berasal dari situs lain. Karena request untuk aksi sensitif biasanya membutuhkan session cookie, pembatasan SameSite yang tepat akan mencegah penyerang mengeksekusi aksi ini secara cross-site. Sejak 2021, Chrome **secara default** menerapkan pembatasan SameSite=Lax. Kemungkinan besar browser lain akan menerapkan perilaku serupa di masa depan.

3. **Referer-based validation**  
   Beberapa aplikasi menggunakan header Referer untuk membela diri terhadap serangan CSRF, dengan **memverifikasi bahwa request** berasal dari domain aplikasi itu sendiri. Namun, ini **umumnya lebih lemah** dibandingkan validasi CSRF token.

Untuk penjelasan lebih detail tentang masing-masing pertahanan, termasuk bagaimana cara **membobol** mekanisme tersebut, lihat materi tambahan berikut (beserta lab interaktifnya):

- **LABS Bypassing CSRF token validation**
- **LABS Bypassing SameSite cookie restrictions**
- **LABS Bypassing Referer-based CSRF defenses**

Untuk detail cara **mengimplementasikan** pertahanan ini dengan benar pada situs Anda, lihat **How to prevent CSRF vulnerabilities**.

---

## XSS vs CSRF
Dalam bagian ini, kami akan menjelaskan **perbedaan antara XSS dan CSRF**, serta membahas apakah CSRF token dapat membantu mencegah serangan XSS.

---

### Apa perbedaan antara XSS dan CSRF?
1. **Cross-site scripting (XSS)** memungkinkan penyerang menjalankan **JavaScript apa saja** di browser korban.  
2. **Cross-site request forgery (CSRF)** memungkinkan penyerang memancing korban agar **melakukan aksi** yang tidak mereka kehendaki.

**Dampak XSS** biasanya **lebih serius** dibandingkan CSRF karena:

- **CSRF** seringkali hanya berlaku untuk beberapa aksi yang bisa dilakukan pengguna. Banyak aplikasi menerapkan pertahanan CSRF secara umum, tetapi mungkin **lupa** pada beberapa aksi tertentu yang dibiarkan rentan.  
- **XSS** dapat membuat penyerang memanfaatkan **seluruh aksi** yang bisa dilakukan pengguna, **tanpa tergantung** pada fungsionalitas spesifik tempat kerentanan terjadi.  
- **CSRF** adalah kerentanan “satu arah” (one-way); penyerang hanya bisa menyuruh korban mengirim request, **tanpa bisa melihat respons**.  
- **XSS** adalah kerentanan “dua arah” (two-way); skrip berbahaya yang disisipkan penyerang **dapat** membaca respons, mencuri data, dan **mengirimkannya kembali** ke domain penyerang.

---

### Apakah CSRF token dapat mencegah serangan XSS?
Beberapa serangan XSS dapat dicegah **dengan penggunaan CSRF token yang efektif**. Misalnya, **reflected XSS sederhana** yang dieksploitasi dengan menaruh payload di URL:

```
https://insecure-website.com/status?message=<script>/*+Kode+jahat...+*/</script>
```

Jika fungsi tersebut memiliki **CSRF token**:

```
https://insecure-website.com/status?csrf-token=CIwNZNlR4XbisJF39I8yWnWX9wX4WFoz&message=<script>/*+Kode+jahat...+*/</script>
```

Dan server benar-benar **memvalidasi** token tersebut, maka token tadi **menghalangi** eksekusi XSS. Sebab, serangan reflected XSS memerlukan request dari **situs lain** (cross-site), dan jika tidak ada token yang valid, request akan **ditolak**.

Namun, ada **beberapa catatan penting**:

1. Jika ada **reflected XSS** di bagian lain situs yang **tidak dilindungi** CSRF token, maka XSS tersebut tetap dapat dieksploitasi.  
2. Jika situs memiliki celah XSS **apa pun**, penyerang bisa menggunakannya untuk melakukan aksi apa saja, bahkan pada fitur yang **dilindungi** token CSRF. Penyerang dapat **meminta** halaman yang berisi token sah, lalu menggunakannya untuk aksi berbahaya.  
3. **CSRF token** tidak melindungi **stored XSS**. Jika sebuah halaman yang dilindungi CSRF token juga memuat konten dari stored XSS, maka XSS tetap bisa dieksekusi secara normal dan payload akan dijalankan saat pengguna mengunjungi halaman tersebut.

---

## Bypassing CSRF token validation
Dalam bagian ini, kami akan menjelaskan **apa itu CSRF token**, bagaimana token tersebut melindungi dari serangan CSRF, dan bagaimana Anda bisa **membobol** pertahanan tersebut.

---

### Apa itu CSRF token?
**CSRF token** adalah **nilai unik, rahasia, dan tidak terduga** yang dihasilkan oleh aplikasi sisi server dan dibagikan ke klien. Saat klien mengirim request untuk melakukan aksi sensitif (misalnya submit form), klien harus menyertakan **CSRF token** yang benar. Jika token tidak cocok, server akan menolak aksi tersebut.

Contoh umum cara server membagikan CSRF token ke klien adalah dengan **menyisipkannya** sebagai field tersembunyi di form HTML:

```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>
```

Saat form tersebut di-submit, request berikut terkirim:

```
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```

**Jika diimplementasikan dengan benar**, CSRF token membantu mencegah CSRF dengan membuat penyerang sulit menyusun request yang valid. Karena penyerang **tidak dapat menebak** nilai token, mereka **tidak bisa** menyertakannya di request jahat.

> **Catatan**  
> CSRF token tidak harus dikirim sebagai parameter tersembunyi di form POST. Beberapa aplikasi meletakkan CSRF token di header HTTP khusus, dan cara pengiriman token juga memengaruhi keamanan keseluruhan. Lihat **How to prevent CSRF vulnerabilities** untuk info lebih lanjut.

---

### Cacat umum dalam validasi CSRF token
Kerentanan CSRF biasanya muncul karena **validasi token yang keliru**. Berikut adalah beberapa kesalahan umum yang memungkinkan penyerang **membypass** pertahanan token CSRF:

#### 1. Validasi CSRF token tergantung pada metode request (POST/GET)
Beberapa aplikasi **memvalidasi token** dengan benar jika request menggunakan metode **POST**, namun **melewatkan validasi** jika request menggunakan **GET**.

Dalam situasi ini, penyerang dapat **beralih ke metode GET** untuk melewati validasi dan melakukan serangan CSRF:

```
GET /email/change?email=pwned@evil-user.net HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```

- **LAB (PRACTITIONER)**: CSRF where token validation depends on request method (Solved)

---

#### 2. Validasi token tergantung pada keberadaan token
Beberapa aplikasi memvalidasi token dengan benar **saat token ada**, tetapi **melewatkan validasi** jika token tidak ada sama sekali.

Dalam situasi ini, penyerang hanya perlu **menghapus parameter token** sepenuhnya untuk membypass validasi:

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net
```

- **LAB (PRACTITIONER)**: CSRF where token validation depends on token being present (Solved)

---

#### 3. CSRF token tidak terikat ke sesi pengguna
Beberapa aplikasi tidak memvalidasi bahwa token benar-benar **milik sesi pengguna** yang mengirimkan request. Aplikasi hanya memeriksa apakah token tersebut **pernah diterbitkan**. 

Dalam situasi ini, penyerang bisa login ke aplikasi dengan akunnya sendiri, mendapatkan **token valid**, lalu memberikan token tersebut ke korban dalam serangan CSRF.

- **LAB (PRACTITIONER)**: CSRF where token is not tied to user session (Solved)

---

#### 4. CSRF token terikat ke cookie non-sesi
Dalam variasi sebelumnya, beberapa aplikasi **mengikat CSRF token** ke sebuah cookie, tapi **cookie tersebut bukan** yang digunakan untuk melacak sesi. Kadang ini terjadi saat aplikasi memakai dua framework berbeda: satu untuk sesi, satu lagi untuk CSRF.

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

Situasi ini **lebih sulit** dieksploitasi, tetapi **masih rentan**. Jika ada fungsi apa pun di situs yang **memungkinkan penyerang** menyetel cookie di browser korban, maka serangan bisa dilakukan. Penyerang login dengan akunnya sendiri, mendapatkan token valid & cookie pendukungnya, lalu mengatur cookie tersebut di browser korban, dan memasukkan token ke request korban.

- **LAB (PRACTITIONER)**: CSRF where token is tied to non-session cookie (Solved)

> **Catatan**  
> Fitur penyetel cookie **tidak harus** ada di aplikasi yang sama. Aplikasi lain di **domain yang sama** (mis. staging.demo.normal-website.com) dapat dipakai untuk menyetel cookie yang berlaku di secure.normal-website.com, jika cakupan cookie itu memungkinkan.

---

#### 5. CSRF token digandakan di dalam cookie
Dalam variasi lain, beberapa aplikasi **tidak menyimpan token di server** sama sekali, melainkan **menduplikasi token** di cookie dan di parameter request. Saat memvalidasi, aplikasi hanya memeriksa apakah nilai token di cookie **sama** dengan parameter request. Ini kadang disebut **“double submit cookie”**.

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

Situasi ini bisa diserang jika situs **memiliki fungsi** yang mengizinkan penyerang menyetel cookie di browser korban. Penyerang tak perlu memperoleh token valid, cukup **mengarang** token baru, menyetel cookie di browser korban, dan memasukkan token itu ke request korban.

- **LAB (PRACTITIONER)**: CSRF where token is duplicated in cookie (Solved)

---

## Bypassing SameSite cookie restrictions
**SameSite** adalah mekanisme keamanan di browser yang menentukan **kapan cookie akan disertakan** dalam request yang berasal dari situs lain. Pembatasan SameSite memberikan **perlindungan parsial** terhadap serangan **cross-site**, termasuk CSRF, cross-site leaks, dan sebagian exploit CORS.

Sejak 2021, **Chrome menerapkan Lax** sebagai default untuk cookie yang tidak memiliki atribut SameSite. Ini adalah **standar yang diusulkan**, dan kami memprediksi **browser lain** akan mengadopsi hal ini juga. Karena itu, Anda perlu memahami bagaimana mekanisme SameSite bekerja dan bagaimana **kemungkinan cara bypass-nya**.

---

### Apa itu “site” dalam konteks SameSite cookies?
Dalam konteks SameSite, **“site”** didefinisikan sebagai **top-level domain (TLD)** (mis. `.com` atau `.net`) **ditambah** satu level domain di depannya (disebut **TLD+1**).

Saat memutuskan apakah suatu request **same-site** atau tidak, **skema URL** (HTTP/HTTPS) juga diperhitungkan. Artinya, tautan dari `http://app.example.com` ke `https://app.example.com` sering dianggap **cross-site** di banyak browser.

> **Catatan**  
> Anda mungkin menjumpai istilah **“effective top-level domain” (eTLD)**. Ini sekadar cara mempertimbangkan sufiks multi-part tertentu yang diperlakukan sebagai TLD, seperti `.co.uk`.

---

### Apa perbedaan “site” dan “origin”?
**Perbedaan** antara “site” dan “origin” adalah **cakupannya**. “Site” bisa mencakup **beberapa domain**, sedangkan “origin” hanya satu. Dua URL dianggap satu origin jika memiliki **skema**, **nama domain**, dan **port** yang sama.

| Request dari             | Request ke                    | Same-site?                                   | Same-origin?                    |
|--------------------------|-------------------------------|----------------------------------------------|---------------------------------|
| https://example.com      | https://example.com          | Ya                                           | Ya                              |
| https://app.example.com  | https://intranet.example.com | Ya (keduanya .example.com)                   | Tidak (domainnya beda)          |
| https://example.com      | https://example.com:8080     | Ya                                           | Tidak (port beda)               |
| https://example.com      | https://example.co.uk        | Tidak (eTLD beda)                            | Tidak (domain beda)             |
| https://example.com      | http://example.com           | Tidak (skema beda)                           | Tidak (skema beda)              |

Penting untuk disadari bahwa **cross-origin** bisa saja **same-site**, namun **tidak** berlaku sebaliknya.

---

### Bagaimana SameSite bekerja?
Sebelum mekanisme SameSite diperkenalkan, browser mengirim cookie **pada setiap request** ke domain yang menerbitkannya, walau request itu dipicu dari situs pihak ketiga. **SameSite** memungkinkan browser dan pemilik situs **membatasi** cross-site request mana yang boleh membawa cookie tertentu.

**SameSite** membantu mengurangi paparan CSRF karena serangan ini biasanya **membutuhkan cookie sesi** korban agar request berbahaya berhasil. Dengan SameSite, browser dapat **menahan cookie** agar tidak dikirim pada request cross-site tertentu.

Mayoritas browser mendukung atribut SameSite dengan nilai:

1. **Strict**  
2. **Lax**  
3. **None**

Pengembang dapat mengonfigurasi level pembatasan SameSite untuk setiap cookie dengan menambahkan atribut di **Set-Cookie**:

```
Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict
```

> **Catatan**  
> Jika situs **tidak menyetel** atribut SameSite, **Chrome** menerapkan **Lax** secara default. Ini adalah **standar baru yang diusulkan** dan kemungkinan akan diadopsi oleh browser lain.

---

#### Strict
Jika cookie disetel dengan `SameSite=Strict`, browser **tidak** akan mengirim cookie tersebut dalam **request cross-site**. Dengan kata lain, jika domain request tidak sama dengan domain yang ditampilkan di address bar, cookie akan **diblokir**.

Ini cocok untuk **cookie sensitif** yang memungkinkan penggunanya memodifikasi data atau melakukan aksi penting lainnya.

---

#### Lax
Untuk **Lax**, browser akan mengirim cookie pada request cross-site **hanya jika**:

- Request menggunakan method **GET**.  
- Request terjadi karena **navigasi level teratas** oleh pengguna (misalnya klik link).

Artinya, cookie **tidak** disertakan pada request **POST** dari situs lain, yang umumnya merupakan aksi yang **mengubah data** (sehingga rawan diserang CSRF).

---

#### None
Dengan `SameSite=None`, **semua pembatasan dinonaktifkan**; browser akan **selalu** mengirim cookie tersebut, termasuk untuk request cross-site. 

Beberapa situs perlu `SameSite=None` karena cookie itu memang dimaksudkan untuk konteks pihak ketiga (mis. cookie tracking). 

Jika Anda melihat cookie dengan `SameSite=None` atau tanpa atribut sama sekali, Anda patut menelusuri apakah cookie tersebut **berguna**. Banyak situs yang menonaktifkan SameSite secara menyeluruh demi **mencegah kerusakan fungsionalitas** yang tidak diinginkan.

> **Penting**  
> Jika Anda menyetel `SameSite=None`, **wajib** juga menambahkan `Secure` (hanya terkirim melalui HTTPS). Jika tidak, browser **akan menolak cookie** tersebut.

---

### Bypassing SameSite Lax restrictions dengan GET request
Dalam praktiknya, server kadang tidak terlalu ketat dalam menerapkan **metode request**. Bahkan jika mereka mengharapkan **POST**, server mungkin **tetap** menerima **GET**. Jika cookie disetel dengan **Lax**, kita mungkin masih bisa **menjalankan CSRF** dengan memanfaatkan GET request yang melibatkan **top-level navigation** (sehingga browser **akan** menyertakan cookie).

Salah satu cara termudah:

```html
<script>
    document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```

Walau aplikasi biasanya mengharapkan POST, beberapa framework punya parameter override (mis. `_method`) yang mendukung metode berbeda:

```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST">
    <input type="hidden" name="_method" value="GET">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```

- **LAB (PRACTITIONER)**: SameSite Lax bypass via method override (Not solved)

---

### Bypassing SameSite restrictions dengan “on-site gadgets”
Jika cookie disetel **Strict**, browser **tidak akan** mengirimkan cookie di request cross-site apa pun. Namun, mungkin ada **“gadget”** di situs yang memicu request **lanjutan** dalam domain yang sama. Contohnya, **client-side redirect** yang menggunakan input dari pengguna untuk membentuk target redirect. 

Bagi browser, redirect semacam ini hanyalah request **same-site** biasa, sehingga semua cookie akan disertakan. Jika Anda bisa **memanipulasi gadget** semacam ini agar melakukan request berbahaya, Anda bisa **sepenuhnya menembus** pembatasan SameSite.

- **LAB (PRACTITIONER)**: SameSite Strict bypass via client-side redirect (Not solved)

> **Catatan**  
> Metode yang sama **tidak** berlaku untuk **server-side redirect**, sebab browser masih menilai request lanjutan sebagai cross-site (berasal dari request awal yang cross-site).

---

### Bypassing SameSite melalui domain “saudara” yang rentan
Ingatlah bahwa **request cross-origin** bisa jadi **same-site** bila domainnya masih dalam cakupan TLD+1 yang sama. Karena itu, Anda perlu **mengaudit** seluruh domain “saudara” (sibling domains). Jika ada kerentanan seperti **XSS**, penyerang dapat menggunakannya untuk request berbahaya ke domain lain **dalam site yang sama**, menembus pertahanan berbasis site.

Selain CSRF, ingat pula bahwa jika situs memakai WebSocket, kemungkinan rentan terhadap **cross-site WebSocket hijacking (CSWSH)**, yaitu CSRF yang menargetkan handshake WebSocket.

- **LAB (PRACTITIONER)**: SameSite Strict bypass via sibling domain (Not solved)

---

### Bypassing SameSite Lax dengan cookie baru
Cookie dengan Lax biasanya tidak disertakan pada POST cross-site, **kecuali** ada **pengecualian** untuk single sign-on (SSO). Chrome tidak menegakkan Lax untuk **120 detik** pertama pada top-level POST request setelah cookie disetel, untuk mencegah **kerusakan SSO**. Artinya, ada **jendela dua menit** di mana pengguna rentan terhadap serangan cross-site.

> **Catatan**  
> Jendela dua menit ini **tidak** berlaku untuk cookie yang **secara eksplisit** disetel `SameSite=Lax`.

Walau sulit menyesuaikan waktu serangan agar pas di jendela itu, Anda bisa menggunakan “gadget” di situs yang **memaksa cookie baru diterbitkan** ke korban, lalu meluncurkan serangan. Contohnya, menjalankan alur login OAuth bisa menyebabkan session baru setiap kali, karena layanan OAuth tidak tahu apakah pengguna masih login di situs target.

Untuk memicu cookie refresh tanpa membuat korban login ulang secara manual, Anda perlu **top-level navigation** agar cookie sesi OAuth korban terkirim. Kemudian, Anda perlu mengarahkan korban kembali ke situs Anda guna melancarkan serangan CSRF berikutnya. Atau, Anda bisa **membuka tab baru** (pop-up) agar tab asli tak terganggu. Browser biasanya memblok pop-up kecuali ada interaksi manual (onclick):

```html
window.onclick = () => {
    window.open('https://vulnerable-website.com/login/sso');
}
```

- **LAB (PRACTITIONER)**: SameSite Lax bypass via cookie refresh (Not solved)

---

## Bypassing Referer-based CSRF defenses
Selain pertahanan dengan CSRF token, beberapa aplikasi menggunakan **header Referer** untuk pertahanan CSRF, biasanya dengan memverifikasi request berasal dari domain aplikasi. Cara ini **kurang efektif** dan sering bisa dibypass.

### Header Referer
Header `Referer` (salah eja dari “Referrer” di spesifikasi HTTP) adalah header **opsional** yang berisi URL halaman web yang menautkan ke sumber yang diminta. Browser umumnya menambahkan header ini secara otomatis. Namun, ada berbagai cara untuk **menghilangkan atau memodifikasi** Referer demi alasan privasi.

---

#### Validasi Referer tergantung pada keberadaan header
Beberapa aplikasi **memvalidasi** nilai Referer dengan benar saat header **ada**, tetapi **melewatkannya** saat header itu **tidak ada**. Dalam situasi ini, penyerang dapat membuat CSRF yang membuat browser korban **membuang** header Referer. Salah satu cara mudah adalah dengan menambahkan meta tag:

```html
<meta name="referrer" content="never">
```

- **LAB (PRACTITIONER)**: CSRF where Referer validation depends on header being present (Not solved)

---

#### Validasi Referer bisa dibypass
Beberapa aplikasi memeriksa domain di Referer dengan cara **naif**, misalnya hanya memeriksa apakah domain Referer **diawali** dengan domain mereka. Penyerang bisa saja membuat subdomain:

```
http://vulnerable-website.com.attacker-website.com/csrf-attack
```

Atau memeriksa apakah string domain aplikasi **terkandung** dalam Referer, penyerang bisa menempatkan domain tersebut di query string:

```
http://attacker-website.com/csrf-attack?vulnerable-website.com
```

> **Catatan**  
> Walau Anda bisa melakukan ini di Burp, dalam praktiknya browser modern sering **memotong** query string dari Referer secara default demi mengurangi risiko kebocoran data sensitif. Anda dapat menimpa perilaku ini dengan header respons:

```
Referrer-Policy: unsafe-url
```

- **LAB (PRACTITIONER)**: CSRF with broken Referer validation (Not solved)

---

## How to prevent CSRF vulnerabilities
Dalam bagian ini, kami akan memberikan **panduan umum** tentang cara **melindungi** situs dari kerentanan CSRF sebagaimana **ditunjukkan** dalam lab-lab sebelumnya.

---

### Gunakan CSRF token
Cara paling **kuat** untuk mencegah CSRF adalah dengan **menyertakan token** dalam request yang relevan. Token tersebut harus:

- **Tidak terduga** (unpredictable) dengan entropi tinggi, sama seperti session token pada umumnya.
- **Terikat** dengan sesi pengguna.
- **Divalidasi** secara ketat **setiap kali** sebelum aksi sensitif dieksekusi.

---

#### Bagaimana CSRF token dihasilkan?
- Token CSRF sebaiknya memiliki entropi tinggi dan bersifat acak (sebaiknya memakai **CSPRNG**).
- Anda bisa menambahkan **user-specific entropy** (mis. ID pengguna atau data sesi) dan mengambil **hash kuat** dari gabungan nilai tersebut.

---

#### Bagaimana CSRF token dikirim?
- **CSRF token** harus diperlakukan sebagai rahasia. Umumnya, token dikirim sebagai **hidden field** di form HTML yang di-submit via **POST**.
- Letakkan field token **sedini mungkin** di dokumen HTML, sebelum field input lain, untuk mengurangi risiko manipulasi HTML yang memudahkan pencurian token.
- Mengirim token via **URL query string** lebih berisiko karena URL **terekam di log**, bisa muncul di **Referer**, dan **terlihat di address bar**.
- Beberapa aplikasi mengirim token di **header HTTP** khusus. Ini menambah pertahanan ekstra karena browser biasanya **tidak** mengizinkan cross-domain request untuk custom header. Tetapi ini membatasi aplikasi ke request berbasis XHR.
- Jangan mengirim CSRF token di dalam **cookie**.

---

#### Bagaimana CSRF token divalidasi?
- Saat token dihasilkan, simpan token di sesi server-side.
- Ketika menerima request sensitif, **periksa** apakah token yang dikirim **cocok** dengan yang disimpan.  
- **Validasi** harus dilakukan pada semua metode HTTP (GET, POST, dsb).  
- Jika request tidak mengandung token, perlakukan sama seperti token yang invalid — **tolak request**.

---

### Gunakan SameSite cookie dengan Strict
Selain token CSRF, sebaiknya **set SameSite** secara eksplisit untuk setiap cookie:

- **SameSite=Strict** untuk memaksimalkan perlindungan CSRF.
- Turunkan ke **Lax** hanya jika diperlukan dan Anda memahami risikonya.
- Hindari **SameSite=None** kecuali Anda mengerti implikasi keamanannya dan membutuhkannya untuk fungsi pihak ketiga.

---

### Waspada serangan cross-origin, same-site
Ingat, meskipun **SameSite** diatur dengan benar, mekanisme ini **tidak** melindungi dari **cross-origin, same-site attacks** (mis. subdomain lain yang rentan XSS). 

Idealnya, **pisahkan konten yang tidak aman** (seperti file unggahan pengguna) ke domain berbeda dari fitur atau data yang sensitif. Saat menguji keamanan, audit juga domain “saudara” yang berada di **site** yang sama.

---
