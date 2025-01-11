# Reflected XSS ke dalam atribut dengan tanda kurung sudut yang di-HTML-encode

Lab ini mengandung kerentanan **reflected cross-site scripting (XSS)** pada fungsi pencarian blog di mana tanda kurung sudut di-HTML-encode. Untuk menyelesaikan lab ini, lakukan serangan cross-site scripting yang menyuntikkan sebuah atribut dan memanggil fungsi `alert`.

---------------------------------------------

Referensi:

- [Contexts dalam Cross-Site Scripting - PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts)

![img](images/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded/1.png)

---------------------------------------------

Ketika kita mencari kata “aaaa”, kata tersebut menjadi nilai dari elemen HTML berikut:

![img](images/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded/2.png)

Dengan payload ini, fungsi `alert` akan dipanggil:

```
" autofocus onfocus=alert(1) x="
```

![img](images/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded/3.png)

Berikut adalah konten HTML yang menjelaskan mengapa payload ini berhasil:

![img](images/Reflected%20XSS%20into%20attribute%20with%20angle%20brackets%20HTML-encoded/4.png)

### Penjelasan Query

Pada contoh di atas, kita memanfaatkan kerentanan **Reflected XSS** dengan menyuntikkan payload ke dalam atribut HTML yang telah di-HTML-encode. Meskipun tanda kurung sudut (`<` dan `>`) telah di-encode, kita masih bisa menyisipkan atribut tambahan untuk memicu eksekusi JavaScript.

Payload yang digunakan adalah:

```
" autofocus onfocus=alert(1) x="
```

**Penjelasan Payload:**

1. **"**: Menutup atribut `value` yang sebelumnya.
2. **autofocus**: Menambahkan atribut `autofocus` ke elemen input, yang akan secara otomatis memfokuskan elemen tersebut saat halaman dimuat.
3. **onfocus=alert(1)**: Menambahkan event handler `onfocus` yang akan menjalankan fungsi `alert(1)` ketika elemen mendapatkan fokus.
4. **x=**: Menambahkan atribut tambahan untuk menjaga struktur HTML tetap valid.

**Proses Eksekusi:**

- Ketika payload disuntikkan, elemen input akan terlihat seperti berikut:

  ```html
  <input type="text" value="" autofocus onfocus=alert(1) x="">
  ```

- Atribut `autofocus` membuat elemen input otomatis mendapatkan fokus saat halaman dimuat.
- Ketika elemen mendapatkan fokus, event `onfocus` dipicu, menjalankan fungsi `alert(1)`.

Dengan demikian, meskipun tanda kurung sudut di-encode, penyisipan atribut tambahan memungkinkan eksekusi JavaScript, mengakibatkan kerentanan XSS.
