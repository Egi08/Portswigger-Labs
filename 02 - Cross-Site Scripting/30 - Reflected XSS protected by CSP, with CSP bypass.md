Tentu! Berikut adalah Proof of Concept (PoC) untuk **Reflected XSS yang dilindungi oleh Content Security Policy (CSP)** dengan **bypass CSP**. Saya juga akan menjelaskan setiap bagian dari query tersebut agar Anda memahami mekanismenya.

---

## **Overview Lab**

- **Jenis Vulnerabilitas:** Reflected Cross-Site Scripting (XSS)
- **Proteksi yang Ada:** Content Security Policy (CSP)
- **Tujuan:** Melakukan XSS dengan mem-bypass CSP dan memanggil fungsi `alert()`
- **Catatan:** Solusi ini khusus berfungsi di **Chrome**

---

## **Langkah-Langkah Solusi**

### 1. **Memasukkan Payload Awal**

**Input di Kotak Pencarian:**

```html
<img src=1 onerror=alert(1)>
```

**Penjelasan:**
- Payload ini mencoba menyisipkan gambar dengan sumber yang tidak valid (`src=1`), sehingga menghasilkan error yang memicu eksekusi `onerror`, yang menjalankan `alert(1)`.
- **Namun**, karena CSP aktif, eksekusi skrip ini diblokir.

### 2. **Menganalisis Header CSP dengan Burp Proxy**

- **Menggunakan Burp Proxy:** Amati respons HTTP yang mengandung header `Content-Security-Policy`.
- **Fokus pada:** `report-uri` yang memiliki parameter `token`.
  
  Contoh Header CSP:

  ```
  Content-Security-Policy: default-src 'self'; report-uri /csp-report?token=abc123
  ```

- **Kunci:** Anda dapat mengontrol parameter `token`, yang memungkinkan injeksi direktif CSP tambahan.

### 3. **Menyuntikkan Direktif CSP Tambahan melalui Parameter `token`**

**URL Target:**

```
https://YOUR-LAB-ID.web-security-academy.net/?search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'
```

**Penjelasan:**

- **`YOUR-LAB-ID`:** Gantilah dengan ID lab Anda.
- **Parameter `search`:** Memasukkan payload XSS yang ingin dieksekusi.
  
  ```html
  <script>alert(1)</script>
  ```

- **Parameter `token`:** Menyuntikkan direktif CSP tambahan.
  
  ```plaintext
  ;script-src-elem 'unsafe-inline'
  ```

- **Mengapa Ini Berfungsi:**
  - **`script-src-elem`:** Direktif CSP yang khusus mengatur sumber skrip untuk elemen `<script>`.
  - **`'unsafe-inline'`:** Mengizinkan eksekusi skrip inline.
  - Dengan menyuntikkan `;script-src-elem 'unsafe-inline'`, Anda **menimpa** aturan CSP sebelumnya untuk `script-src-elem`, memungkinkan eksekusi skrip inline yang sebelumnya diblokir.

### 4. **URL Final PoC**

```
https://YOUR-LAB-ID.web-security-academy.net/?search=<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'
```

**Cara Kerja:**

1. **Permintaan (Request):**
   - Browser mengirimkan permintaan dengan parameter `search` berisi payload XSS dan `token` berisi injeksi CSP.
   
2. **Respons dari Server:**
   - Header `Content-Security-Policy` menjadi:
     
     ```
     Content-Security-Policy: default-src 'self'; report-uri /csp-report?token=;script-src-elem 'unsafe-inline'
     ```
   
   - Injeksi `;script-src-elem 'unsafe-inline'` menimpa aturan sebelumnya.

3. **Eksekusi Skrip:**
   - Dengan `script-src-elem 'unsafe-inline'`, browser mengizinkan eksekusi skrip inline, sehingga payload `<script>alert(1)</script>` berhasil dieksekusi, memunculkan alert.

---

## **Diagram Alur**

1. **Pengguna** memasukkan payload XSS ke dalam parameter `search`.
2. **Parameter `token`** disuntikkan dengan direktif CSP tambahan.
3. **Server** mengembalikan halaman dengan header CSP yang telah dimodifikasi.
4. **Browser** menerima header CSP baru yang mengizinkan eksekusi skrip inline.
5. **Skrip** dalam parameter `search` dieksekusi, memicu `alert(1)`.

---

## **Catatan Penting**

- **Etika dan Legalitas:**
  - **Hanya lakukan pengujian** pada aplikasi yang Anda miliki izin eksplisit untuk diuji.
  - **Jangan pernah mencoba** teknik ini pada situs web tanpa izin, karena ini melanggar hukum dan etika profesional.

- **Pemahaman CSP:**
  - **Content Security Policy (CSP)** adalah mekanisme keamanan yang membantu mencegah berbagai jenis serangan, termasuk XSS.
  - **Bypassing CSP** seperti di atas menunjukkan betapa pentingnya konfigurasi CSP yang ketat dan validasi input yang baik.

- **Keamanan Tambahan:**
  - **Validasi dan Sanitasi Input:** Pastikan semua input pengguna divalidasi dan disanitasi dengan benar di sisi server.
  - **Gunakan CSP yang Kuat:** Hindari penggunaan `'unsafe-inline'` dan minimalkan sumber skrip eksternal.
  - **Pemantauan dan Pelaporan:** Gunakan direktif seperti `report-uri` atau `report-to` untuk memantau potensi pelanggaran CSP.

---

## **Kesimpulan**

Dengan memahami cara kerja CSP dan teknik bypass seperti di atas, Anda dapat lebih baik dalam mengamankan aplikasi web dari serangan XSS. Selalu prioritaskan praktik pengkodean yang aman dan konfigurasi keamanan yang tepat untuk melindungi aplikasi dan penggunanya.

Jika Anda memiliki pertanyaan lebih lanjut atau membutuhkan klarifikasi tambahan, jangan ragu untuk bertanya!
