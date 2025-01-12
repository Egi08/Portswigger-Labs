### **Proof of Concept (PoC) Reflected XSS dengan Event Handlers dan Atribut `href` Diblokir**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada lab ini, beberapa tag HTML diizinkan (whitelisted), namun semua event handlers (seperti `onclick`) dan atribut `href` pada tag `<a>` diblokir. Tantangan ini adalah menyuntikkan vektor yang, ketika diklik, memanggil fungsi `alert` tanpa menggunakan event handlers atau atribut `href`.

#### **Langkah-langkah PoC**

##### **1. Memahami Pembatasan yang Diberikan**
- **Tag yang Diizinkan:** Beberapa tag seperti `<a>`, `<svg>`, dan lainnya diizinkan.
- **Event Handlers dan Atribut `href` Diblokir:** Tidak dapat menggunakan atribut seperti `onclick`, `onmouseover`, atau `href="javascript:alert(1)"`.
- **Tujuan:** Menyisipkan elemen yang dapat diklik oleh pengguna dan memanggil fungsi `alert` tanpa menggunakan event handlers atau atribut `href`.

##### **2. Menyusun Payload yang Efektif**
Karena event handlers dan atribut `href` diblokir, kita perlu mencari cara alternatif untuk mengeksekusi JavaScript. Salah satu metode yang efektif adalah memanfaatkan elemen SVG dengan tag `<animate>` yang dapat memodifikasi atribut secara dinamis.

- **Payload:**

  ```html
  <svg><a><animate attributeName="href" from="javascript:alert(1)" to="javascript:alert(1)" /></a>Click</svg>
  ```

- **Penjelasan Payload:**
  - `<svg>`: Tag SVG yang diizinkan.
  - `<a>`: Tag anchor yang diizinkan, meskipun atribut `href` diblokir secara langsung.
  - `<animate>`: Elemen SVG yang digunakan untuk menganimasi atribut. Di sini, digunakan untuk menyetel atribut `href` ke `javascript:alert(1)` tanpa harus menulisnya secara eksplisit.
  - `Click`: Label yang akan dilihat oleh pengguna dan dapat diklik.

##### **3. Meng-URL Encode Payload**
Agar payload dapat disisipkan ke dalam parameter URL, payload di atas perlu di-URL encode terlebih dahulu.

- **Encoded Payload:**

  ```
  %3Csvg%3E%3Ca%3E%3Canimate%20attributeName%3D%22href%22%20from%3D%22javascript%3Aalert%281%29%22%20to%3D%22javascript%3Aalert%281%29%22%20/%3E%3C/a%3EClick%3C/svg%3E
  ```

##### **4. Menyisipkan Payload melalui Burp Suite Intruder**
Berikut adalah langkah-langkah untuk menyisipkan payload menggunakan Burp Suite Intruder:

1. **Menyiapkan Intruder:**
   - Buka **Burp Suite** dan aktifkan proxy.
   - Pastikan browser Anda dikonfigurasi untuk menggunakan proxy Burp Suite.
   
2. **Mengirim Permintaan ke Intruder:**
   - Kirimkan permintaan pencarian dengan payload kosong atau placeholder, misalnya:
     
     ```
     GET /?search=<§§> HTTP/1.1
     Host: 0a390097047a41468253107000710099.web-security-academy.net
     Cookie: session=Yhmn97WwdLFLSsRl08Yr9kX96uQPmI8l
     (Header lainnya...)
     ```

   - Di Intruder, tentukan posisi payload dengan menandai `§§` sebagai titik injeksi.
   
3. **Memilih Payload:**
   - Buka [Cross-Site Scripting (XSS) Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) untuk referensi berbagai vektor XSS.
   - Cari payload yang relevan dengan situasi ini, misalnya yang menggunakan elemen SVG dan `<animate>`.
   - Masukkan payload yang telah di-URL encode ke dalam Intruder sebagai payload list.

4. **Menjalankan Serangan:**
   - Jalankan Intruder dan biarkan alat mengirimkan berbagai payload.
   - Observasi respons dari server untuk menemukan payload yang berhasil dieksekusi (munculnya alert).

##### **5. Mengirimkan Payload melalui URL Lab**
Setelah menemukan payload yang berhasil melalui Intruder, Anda dapat menyusun URL eksploitasi langsung.

- **Contoh URL Eksploitasi:**

  ```
  https://YOUR-LAB-ID.web-security-academy.net/?search=%3Csvg%3E%3Ca%3E%3Canimate%20attributeName%3D%22href%22%20from%3D%22javascript%3Aalert%281%29%22%20to%3D%22javascript%3Aalert%281%29%22%20/%3E%3C/a%3EClick%3C/svg%3E
  ```

  - **Ganti `YOUR-LAB-ID`** dengan ID lab Anda.

##### **6. Memverifikasi Eksekusi Payload**
- **Langkah-langkah:**
  1. **Salin URL Eksploitasi:**
     - Pastikan URL yang telah disusun dengan payload benar.
     
  2. **Buka Browser (Chrome):**
     - Buka browser Chrome dan tempelkan URL yang telah disalin.
     
  3. **Klik "Click":**
     - Setelah halaman dimuat, klik pada teks "Click" yang disisipkan.
     - Sebuah popup alert dengan pesan `1` akan muncul, menandakan bahwa serangan XSS berhasil dieksekusi.

  - **Gambar Ilustrasi:**
  
    ![Eksekusi Payload](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/1.png)

#### **Mengapa Payload Ini Berfungsi?**
Meskipun event handlers seperti `onclick` dan atribut `href` diblokir secara langsung, payload ini memanfaatkan kemampuan SVG untuk menganimasi atribut yang secara tidak langsung dapat memicu eksekusi JavaScript. Dengan menggunakan elemen `<animate>`, atribut `href` pada tag `<a>` diatur ke `javascript:alert(1)` tanpa harus menulisnya secara eksplisit, sehingga melewati filter yang memblokir atribut tersebut.

1. **Menggunakan SVG `<animate>`:**
   - Elemen `<animate>` dalam SVG dapat digunakan untuk mengubah atribut secara dinamis. Dalam payload ini, `attributeName="href"` diubah dari `javascript:alert(1)` ke `javascript:alert(1)`, yang sebenarnya hanya menetapkan ulang nilai yang sama. Namun, ini cukup untuk memicu eksekusi JavaScript ketika elemen `<a>` diklik.

2. **Label "Click":**
   - Dengan memberikan label "Click" pada elemen `<a>`, pengguna terdorong untuk mengkliknya, yang kemudian menjalankan fungsi `alert(1)` melalui atribut `href` yang telah diubah.

#### **Catatan Penting**
- **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif seperti cookie sesi, melakukan pengambilalihan akun, atau mengubah tampilan dan fungsi halaman web.
  
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
  
- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (**Chrome**) karena perilaku penanganan XSS dan SVG dapat berbeda antar browser.

#### **Referensi:**
- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)
- [Cross-Site Scripting (XSS) Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

---

![Langkah 1: Pembatasan yang Diberikan](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/1.png)
![Langkah 2: Mengirim Payload](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/2.png)
![Langkah 3: Eksekusi Payload](images/Reflected%20XSS%20with%20event%20handlers%20and%20href%20attributes%20blocked/3.png)
