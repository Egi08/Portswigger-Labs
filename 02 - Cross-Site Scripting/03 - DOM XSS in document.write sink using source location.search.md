
# DOM XSS in document.write sink using source location.search

This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the alert function.

---------------------------------------------

References:

- https://portswigger.net/web-security/cross-site-scripting/dom-based

---------------------------------------------

---

### **Langkah 1: Masukkan Input Bebas**
1. **Masukkan teks sembarang** (contohnya: `aaa`) di kolom pencarian atau input form pada aplikasi web.
2. Klik tombol **Cari** atau submit untuk mengirimkan data ke server.

---

### **Langkah 2: Analisis Melalui Inspect Element**
1. **Buka Developer Tools** di browser Anda dengan cara klik kanan pada halaman dan pilih **Inspect** atau tekan tombol `Ctrl+Shift+I` (Windows/Linux) atau `Cmd+Option+I` (Mac).
2. Cari **teks "aaa"** yang telah dimasukkan tadi.
   - Gunakan fitur pencarian di Developer Tools dengan menekan `Ctrl+F` dan ketikkan `aaa` untuk menemukan di mana teks tersebut ditampilkan di HTML.
3. Analisis di mana teks tersebut dimuat:
   - Apakah dimasukkan dalam atribut HTML seperti `img src`, `href`, atau `value`.
   - Atau ditampilkan langsung di dalam elemen HTML seperti `<div>`, `<p>`, atau `<span>`.

---

### **Langkah 3: Masukkan Payload XSS**
Setelah menemukan lokasi di mana input "aaa" dimasukkan, coba masukkan payload berikut untuk menguji kemungkinan XSS:

#### **Payload Sederhana:**
```html
"><svg onload=alert(1)>
```

#### **Penjelasan Payload:**
- **`">`**: Menutup atribut atau elemen HTML yang sebelumnya (misalnya `src="`).
- **`<svg onload=alert(1)>`**: Memasukkan elemen SVG dengan atribut `onload`, yang langsung menjalankan fungsi JavaScript `alert(1)`.

---

### **Langkah 4: Observasi**
1. Setelah payload dikirimkan, perhatikan apakah **popup alert** dengan angka "1" muncul.
   - Jika popup muncul, berarti aplikasi rentan terhadap **reflected XSS**.
   - Jika tidak muncul, kemungkinan aplikasi memiliki mekanisme sanitasi input atau validasi.

---

### **Catatan Tambahan**
1. **Target Lokasi Input XSS:**
   - Jika teks "aaa" muncul dalam atribut seperti `src`, `href`, atau `alt`, maka payload akan dimasukkan ke sana.
   - Jika muncul langsung dalam elemen HTML (misalnya `<div>` atau `<p>`), coba payload berikut:
     ```html
     <script>alert(1)</script>
     ```

2. **Filter atau Sanitasi:**
   - Jika payload tidak dieksekusi, aplikasi mungkin memiliki filter atau mekanisme sanitasi. Untuk bypass, coba payload lain atau enkode karakter khusus.

3. **Tes Payload Lebih Kompleks:**
   - Jika payload sederhana tidak berhasil, gunakan payload XSS yang lebih kompleks, seperti:
     ```html
     "><img src=x onerror=alert(1)>
     ```

---

### **Kesimpulan**
Proses ini membantu mengidentifikasi kerentanan XSS di aplikasi web. Langkah utama melibatkan memasukkan teks biasa, mencari bagaimana teks tersebut diproses di HTML, dan mencoba payload untuk memicu eksekusi JavaScript di browser.
