### **Eksploitasi DOM XSS pada jQuery Selector Sink Menggunakan Hashchange Event**

#### **Tujuan Lab:**
Tujuan dari lab ini adalah mengeksploitasi kerentanan **DOM-based XSS** di halaman utama dengan menyisipkan payload yang memicu fungsi `print()` di browser korban.

---

### **Analisis Kerentanan**
1. **Kode yang Bermasalah:**
   ```javascript
   $(window).on('hashchange', function() {
       var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
       if (post) post.get(0).scrollIntoView();
   });
   ```
   - Kode ini mendengarkan event `hashchange`.
   - Properti `window.location.hash` digunakan sebagai input ke dalam jQuery `:contains` selector tanpa sanitasi.
   - Hal ini memungkinkan penyisipan string berbahaya yang dapat menyebabkan DOM-based XSS.

2. **Vektor Eksploitasi:**
   - Payload berbahaya dapat disuntikkan ketika hash dalam URL diubah.
   - **iframe** digunakan untuk mengontrol URL hash dan menyisipkan payload.

---

### **Payload untuk Eksploitasi**
Gunakan payload berikut untuk mengeksploitasi kerentanan:

```html
<iframe style="width:100%;height:100%" src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

#### **Penjelasan Payload:**
- **`iframe`:** Menyisipkan aplikasi yang rentan ke dalam exploit server.
- **`src="https://YOUR-LAB-ID.web-security-academy.net/#"`:** Menunjukkan URL aplikasi yang rentan, dengan menambahkan `#` untuk memicu event `hashchange`.
- **`onload="this.src+='<img src=x onerror=print()>'"`:**
  - Ketika iframe dimuat, atribut `onload` memodifikasi properti `src` untuk menambahkan hash berbahaya yang mengandung tag `<img>`.
  - Tag `<img>` memicu event `onerror` yang menjalankan fungsi `print()`.

---

### **Langkah-Langkah Eksploitasi**

1. **Identifikasi Kode Rentan:**
   - Gunakan DevTools browser atau Burp Suite untuk memeriksa kode yang menangani event `hashchange`.
   - Pastikan bahwa `window.location.hash` digunakan tanpa sanitasi dalam jQuery `:contains` selector.

2. **Buka Exploit Server:**
   - Di antarmuka lab, klik **Go to exploit server**.

3. **Buat Exploit:**
   - Di bagian **Body** exploit server, tempelkan payload berikut:
     ```html
     <iframe style="width:100%;height:100%" src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
     ```
   - Ganti `YOUR-LAB-ID` dengan domain lab yang diberikan.

4. **Simpan dan Kirim Eksploitasi:**
   - Klik **Store** untuk menyimpan exploit.
   - Klik **View exploit** untuk memverifikasi bahwa fungsi `print()` dipanggil di browser Anda.
   - Kembali ke exploit server, lalu klik **Deliver to victim** untuk menyelesaikan lab.

---

### **Verifikasi**
Setelah payload berhasil dikirim, sistem lab akan mengonfirmasi bahwa fungsi `print()` telah dipanggil di browser korban, dan lab akan ditandai sebagai selesai.

---

### **Referensi Gambar**
1. **Menganalisis Kode Rentan:**
   ![Kode Rentan](images/DOM%20XSS%20in%20jQuery%20selector%20sink%20using%20a%20hashchange%20event/1.png)

2. **Membuat Payload di Exploit Server:**
   ![Payload Exploit](images/DOM%20XSS%20in%20jQuery%20selector%20sink%20using%20a%20hashchange%20event/2.png)

---

### **Catatan Penting:**
- Pastikan Anda mengganti `YOUR-LAB-ID` dalam payload dengan domain lab yang sesuai.
- Contoh ini menggunakan fungsi `print()` untuk tujuan demonstrasi. Dalam skenario nyata, kerentanan ini dapat dimanfaatkan untuk serangan lebih berbahaya, seperti mencuri informasi sensitif.
