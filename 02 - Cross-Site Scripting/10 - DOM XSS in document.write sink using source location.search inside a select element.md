
# DOM XSS in document.write sink using source location.search inside a select element
# DOM XSS pada Sink `document.write` Menggunakan `location.search` di dalam Elemen Select

Lab ini mengandung kerentanan **DOM-based cross-site scripting (XSS)** pada fungsi pengecekan stok. Kerentanan ini memanfaatkan fungsi JavaScript `document.write`, yang menuliskan data ke halaman. Fungsi `document.write` dipanggil dengan data dari `location.search` yang dapat Anda kendalikan menggunakan URL situs web. Data tersebut ditempatkan di dalam elemen `<select>`.

Untuk menyelesaikan lab ini, lakukan serangan cross-site scripting yang keluar dari elemen `<select>` dan memanggil fungsi `alert`.

---------------------------------------------

**Referensi:**

- [DOM-based Cross-Site Scripting - PortSwigger](https://portswigger.net/web-security/cross-site-scripting/dom-based)

---------------------------------------------

Sink yang digunakan adalah:

```javascript
var stores = ["London","Paris","Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
if(store) {
    document.write('<option selected>'+store+'</option>');
}
for(var i=0;i<stores.length;i++) {
    if(stores[i] === store) {
        continue;
    }
    document.write('<option>'+stores[i]+'</option>');
}
document.write('</select>');
```

![img](images/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element/1.png)

Parameter `storeId` dituliskan di antara tag `<option selected>` dan `</option>`. Artinya, jika kita menambahkan nilai tersebut dalam permintaan GET, nilai tersebut akan muncul di antara opsi-opsi, misalnya mengakses `/product?productId=4&storeId=1`:

![img](images/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element/2.png)

Untuk keluar dari tag `<option>`, kita dapat menggunakan payload berikut:

```
</option><script>alert(1)</script><option selected>
```

Sehingga URL yang dihasilkan menjadi:

```
/product?productId=4&storeId=</option><script>alert(1)</script><option%20selected>
```

![img](images/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element/3.png)

Dengan payload ini, fungsi `alert` akan dijalankan saat link diklik:

![img](images/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element/4.png)

### Penjelasan Query

Pada contoh di atas, kita memanfaatkan kerentanan **DOM-based XSS** dengan menyuntikkan payload ke dalam string yang dituliskan oleh fungsi `document.write` di dalam elemen `<select>`. Meskipun tanda kurung sudut (`<` dan `>`) di-HTML encode, kita masih dapat memecahkan struktur HTML dan menyisipkan kode JavaScript berbahaya.

**Payload yang digunakan adalah:**

```
</option><script>alert(1)</script><option selected>
```

**Penjelasan Payload:**

1. **`</option>`**: Menutup tag `<option>` yang sedang dibuka, sehingga keluar dari elemen `<select>`.
2. **`<script>alert(1)</script>`**: Menyisipkan elemen `<script>` yang menjalankan fungsi `alert(1)`.
3. **`<option selected>`**: Membuka kembali elemen `<option>` dengan atribut `selected` untuk menjaga struktur HTML tetap valid dan mencegah terjadinya error yang dapat menghentikan eksekusi JavaScript.

**Proses Penyisipan dan Eksekusi:**

1. **Menyisipkan Payload:**
   - Saat memasukkan parameter `storeId` dalam URL, masukkan payload `</option><script>alert(1)</script><option selected>`.
   - Meskipun tanda kurung sudut di-encode, penyisipan ini memungkinkan kita untuk keluar dari elemen `<select>` dan menyisipkan elemen `<script>` yang menjalankan JavaScript.

2. **Kode HTML yang Dihasilkan:**
   - Sebelum penyisipan payload, kode HTML yang dihasilkan adalah:
     ```html
     <select name="storeId">
         <option selected>1</option>
         <option>London</option>
         <option>Paris</option>
         <option>Milan</option>
     </select>
     ```
   - Setelah penyisipan payload, kode HTML menjadi:
     ```html
     <select name="storeId">
         </option><script>alert(1)</script><option selected></option>
         <option>London</option>
         <option>Paris</option>
         <option>Milan</option>
     </select>
     ```

3. **Eksekusi JavaScript:**
   - Browser akan memproses elemen `<script>alert(1)</script>` dan menjalankan fungsi `alert(1)`, menampilkan kotak dialog alert dengan pesan `1`.

**Mengapa Payload Berhasil:**

- **Pemecahan Struktur HTML:** Dengan menyisipkan `</option>`, kita dapat keluar dari elemen `<option>` yang sedang dibuka dan memasukkan elemen HTML baru seperti `<script>`.
- **Sintaks HTML yang Valid:** Dengan menambahkan `<option selected>`, kita memastikan bahwa struktur HTML tetap valid, sehingga mencegah error yang dapat menghentikan eksekusi JavaScript.
- **DOM-based XSS:** Payload disisipkan ke dalam DOM dan dieksekusi oleh browser tanpa disanitasi yang memadai, memungkinkan eksekusi kode JavaScript berbahaya.

**Langkah-langkah Penyelesaian Lab:**

1. **Modifikasi URL:**
   - Akses URL dengan parameter `storeId` yang telah disisipi payload:
     ```
     /product?productId=4&storeId=</option><script>alert(1)</script><option%20selected>
     ```

2. **Verifikasi Eksekusi:**
   - Setelah halaman dimuat, fungsi `alert(1)` akan dijalankan secara otomatis, menampilkan kotak dialog alert dengan pesan `1`.

Dengan demikian, meskipun tanda kurung sudut di-HTML encode, kemampuan untuk memecahkan struktur HTML dan menyisipkan elemen `<script>` memungkinkan eksekusi kode JavaScript, menyebabkan kerentanan DOM-based XSS.(images/DOM%20XSS%20in%20document.write%20sink%20using%20source%20location.search%20inside%20a%20select%20element/3.png)
