---

# Reflected XSS Dilindungi oleh CSP, dengan Bypass CSP | Apr 27, 2025

## **Pendahuluan**

Selamat datang di write-up saya yang lainnya! Dalam lab PortSwigger ini, Anda akan mempelajari: **Reflected XSS yang dilindungi oleh Content Security Policy (CSP), dengan bypass CSP**. Tanpa berlama-lama lagi, mari kita mulai.

**Tingkat Kesulitan secara Keseluruhan untuk Saya (Dari 1-10 bintang):** ★★★☆☆☆☆☆☆☆

## **Latar Belakang**

Lab ini menggunakan CSP dan mengandung kerentanan Reflected Cross-Site Scripting (XSS).

Untuk menyelesaikan lab ini, lakukan serangan XSS yang dapat melewati CSP dan memanggil fungsi `alert`.

Harap dicatat bahwa solusi yang dimaksud untuk lab ini hanya memungkinkan di **Chrome**.

## **Eksploitasi**

### **Halaman Utama:**

![Halaman Utama](#)

Di sini, kita dapat melihat terdapat kotak pencarian.

### **Melakukan Pencarian:**

![Melakukan Pencarian](#)

Seperti yang Anda lihat, input kita direfleksikan kembali ke halaman web.

### **Sejarah HTTP di Burp Suite:**

![Sejarah HTTP Burp Suite](#)

Kita dapat melihat bahwa CSP (Content Security Policy) diaktifkan:

```
Content-Security-Policy: default-src 'self'; object-src 'none'; script-src 'self'; style-src 'self'; report-uri /csp-report?token=
```

Perhatikan bahwa `script-src` diatur ke `self`, yang berarti hanya mengizinkan JavaScript dimuat dari origin yang sama dengan halaman itu sendiri.

Namun, kita juga dapat melihat terdapat direktif `report-uri`, yang merefleksikan input ke dalam kebijakan CSP aktual.

Jika situs merefleksikan parameter yang dapat kita kontrol, kita dapat menyuntikkan titik koma (`;`) untuk menambahkan direktif CSP kita sendiri.

Biasanya, tidak mungkin untuk menimpa direktif `script-src` yang sudah ada. Namun, Chrome memperkenalkan direktif `script-src-elem`, yang memungkinkan Anda mengontrol elemen skrip, tetapi tidak event. Pentingnya, direktif baru ini memungkinkan Anda untuk menimpa direktif `script-src` yang sudah ada.

Dengan informasi di atas, kita dapat mencoba untuk melewati CSP dengan menyuntikkan kebijakan CSP baru.

### **Payload:**

```
/?token=;script-src-elem 'unsafe-inline'&search=<script>alert(document.domain)</script>
```

**Catatan:** Jika Anda tidak melihat banner yang lengkap, atur `script-src-elem` ke `none`.

**Bagus!**

## **Apa yang Telah Kita Pelajari:**

- **Reflected XSS** yang dilindungi oleh **CSP**, dengan teknik **bypass CSP**.
- Pentingnya memahami bagaimana **CSP** bekerja dan bagaimana kebijakan ini dapat dieksploitasi jika ada celah dalam implementasinya.
- **Chrome** memiliki dukungan khusus yang memungkinkan bypass CSP melalui direktif `script-src-elem`.
- Pentingnya **validasi dan sanitasi input** yang tepat untuk mencegah serangan XSS.
- Pentingnya konfigurasi **CSP** yang kuat dan spesifik untuk melindungi aplikasi web dari berbagai jenis serangan skrip.

---

## **Diagram Alur**

1. **Pengguna** memasukkan payload XSS ke dalam parameter `search`.
2. **Parameter `token`** disuntikkan dengan direktif CSP tambahan.
3. **Server** mengembalikan halaman dengan header CSP yang telah dimodifikasi.
4. **Browser** menerima header CSP baru yang mengizinkan eksekusi skrip inline.
5. **Skrip** dalam parameter `search` dieksekusi, memicu `alert(document.domain)`.

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
