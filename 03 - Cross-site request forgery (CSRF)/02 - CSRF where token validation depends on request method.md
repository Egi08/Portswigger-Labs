
# CSRF where token validation depends on request method

Fitur perubahan email pada lab ini **rentan terhadap serangan CSRF**. Aplikasi tersebut **mencoba** memblokir serangan CSRF, tetapi **hanya** menerapkan pertahanan pada tipe request tertentu saja.

**Untuk menyelesaikan lab**, kita perlu menggunakan exploit server untuk menampilkan halaman HTML yang menjalankan serangan CSRF guna mengubah alamat email korban.

Anda bisa login ke akun sendiri menggunakan kredensial berikut:
```
Username: wiener
Password: peter
```

> **Hint**: Anda tidak dapat mendaftarkan email yang sudah digunakan pengguna lain. Jika Anda mengubah alamat email akun Anda sendiri selama uji coba, pastikan menggunakan email lain saat menyerahkan exploit final ke korban.

---

### Referensi
- [PortSwigger - CSRF](https://portswigger.net/web-security/csrf)

---

### Tautan yang dihasilkan
```
https://0af7002903ddc72c818c574600ce0059.web-security-academy.net/
```

---

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/1.png)

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/2.png)

Pada dasarnya, formulir menggunakan **request POST**:

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/3.png)

Namun, **ubah method** menjadi **GET** seperti berikut:

```
/my-account/change-email?email=test3%40test.com
```

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/4.png)

Anda akan mendapatkan **kode redirect**. Jika diikuti, **alamat email** berhasil diupdate:

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/5.png)

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/6.png)

### Payload
Anda bisa menggunakan **payload** berikut di halaman HTML Anda:

```
<body onload="window.open('https://0af7002903ddc72c818c574600ce0059.web-security-academy.net/my-account/change-email?email=test3%40test.com')">
```

![img](images/CSRF%20where%20token%20validation%20depends%20on%20request%20method/7.png)

---

## Penjelasan Singkat PoC
1. **Method GET** ternyata tidak dilindungi token CSRF di server, sehingga request GET bisa dieksploitasi.
2. Dengan menempatkan payload `<body onload="window.open('...')">`, browser korban akan otomatis **membuka URL** yang memicu perubahan email.
3. Ketika korban mengunjungi halaman exploit, **sesi login mereka (jika ada)** akan digunakan oleh browser untuk mengirim request GET, sehingga email korban berubah menjadi `test3@test.com` (atau alamat lain yang ditentukan).

**Langkah akhir**: Unggah HTML PoC tersebut ke **exploit server** Anda, kemudian berikan link ke korban. Saat korban membukanya dan masih login, email mereka akan diubah sesuai value yang Anda masukkan.
