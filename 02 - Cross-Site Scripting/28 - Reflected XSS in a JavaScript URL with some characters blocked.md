### Penjelasan POC (Proof of Concept)
**Lab: Reflected XSS in a JavaScript URL with some characters blocked**

Lab ini melibatkan serangan XSS (Cross-Site Scripting) yang memanfaatkan kerentanan pada URL berbasis JavaScript. Tantangannya adalah beberapa karakter tertentu diblokir untuk mencegah serangan XSS, sehingga Anda perlu menemukan cara untuk menyiasatinya agar dapat menjalankan fungsi `alert` dengan string `1337` di dalam pesan.

**Skenario:**  
- Input Anda akan direfleksikan ke dalam JavaScript URL.  
- Sebagian karakter seperti spasi diblokir, sehingga Anda harus menggunakan metode kreatif untuk mem-bypass pembatasan ini.  

Tujuannya adalah menampilkan pop-up alert dengan angka `1337`.

---

### Penjelasan Query
URL eksploit yang diberikan adalah sebagai berikut:  
```plaintext
https://YOUR-LAB-ID.web-security-academy.net/post?postId=5&%27},x=x=%3E{throw/**/onerror=alert,1337},toString=x,window%2b%27%27,{x:%27
```

**Penjelasan Komponen Query:**
1. **`throw/**/onerror=alert,1337`**  
   - `throw` digunakan untuk memicu kesalahan, dan komentar `/**/` digunakan sebagai pengganti spasi.  
   - `onerror=alert,1337` mengaitkan fungsi `alert` dengan `onerror`, sehingga `alert('1337')` akan dipanggil saat terjadi kesalahan.

2. **`x=x=>{}`**  
   - Digunakan untuk mendefinisikan fungsi panah (arrow function).  
   - Fungsi ini mengelola blok kode, memungkinkan penggunaan `throw` di dalamnya.

3. **`toString=x`**  
   - Properti `toString` diatur ke fungsi `x` yang didefinisikan sebelumnya.

4. **`window+''`**  
   - Memaksa konversi objek `window` ke string, yang akan memanggil fungsi `toString`.

5. **`{x:'}`**  
   - Blok ini memastikan format input tetap valid sesuai sintaks JavaScript.

---

### Cara Menggunakan:
1. Masukkan URL dengan mengganti `YOUR-LAB-ID` sesuai dengan ID lab Anda.
2. Buka halaman tersebut dan klik tombol "Back to blog" di bagian bawah halaman.
3. Jika berhasil, pop-up alert dengan angka `1337` akan muncul.

---

### Kesimpulan:
Eksploit ini memanfaatkan kombinasi:
- Penggunaan fungsi panah untuk mengelola blok.
- Properti `toString` pada objek `window`.
- Pemisah karakter dengan komentar untuk menggantikan spasi.  
Teknik ini menunjukkan bagaimana kode JavaScript dapat dimanipulasi meskipun ada pembatasan karakter.
