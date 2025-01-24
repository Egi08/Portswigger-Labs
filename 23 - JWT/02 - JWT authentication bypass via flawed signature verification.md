# JWT authentication bypass melalui verifikasi tanda tangan yang cacat

Lab ini menggunakan mekanisme berbasis JWT untuk menangani sesi. Server dikonfigurasi dengan tidak aman sehingga menerima JWT tanpa tanda tangan.

Untuk menyelesaikan lab ini, modifikasi token sesi Anda untuk mendapatkan akses ke panel admin di /admin, lalu hapus pengguna carlos.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: wiener:peter

**Tips:** Kami merekomendasikan untuk membiasakan diri dengan cara kerja JWT di Burp Suite sebelum mencoba lab ini.

---------------------------------------------

**Referensi:**  
- [JWT Authentication Bypass](https://portswigger.net/web-security/jwt)

---------------------------------------------

### Bukti Konsep (Proof of Concept - POC)

1. **Login dengan akun wiener:peter**  
   ![image](https://github.com/user-attachments/assets/b3e4811a-e5fb-4212-bbc3-bf798822e454)  
   Masuk ke aplikasi menggunakan kredensial yang diberikan untuk mendapatkan token sesi awal.

2. **Buka OWASP Penetration Testing Kit**  
   ![image](https://github.com/user-attachments/assets/1a631c81-ef99-4c49-9a7d-f12454db6ec4)  
   Gunakan OWASP Penetration Testing Kit atau alat seperti Burp Suite untuk menganalisis dan memodifikasi token JWT.

3. **Ubah Algorithm `RS256` menjadi `none` lalu ubah juga sub menjadi `administrator` dan perbarui di semua sumber**  
   ![image](https://github.com/user-attachments/assets/51c8c2a9-9563-43cd-bad0-d56ccad7d27a)  
   ![image](https://github.com/user-attachments/assets/83bfb227-9cc5-4427-87ed-b7ca0e7ca0ac)  
   Modifikasi algoritma JWT dari `RS256` menjadi `none`, yang menunjukkan bahwa tidak ada tanda tangan yang diperlukan. Selain itu, ubah nilai `sub` menjadi `administrator` untuk mendapatkan hak akses admin.

4. **Muat ulang web untuk menampilkan panel admin**  
   ![image](https://github.com/user-attachments/assets/de8c14dd-5605-49bf-8407-65e4a1c5b29d)  
   Setelah token diperbarui, muat ulang halaman. Panel admin sekarang dapat diakses.

5. **Hapus pengguna carlos**  
   ![image](https://github.com/user-attachments/assets/55b85770-3be3-4e0f-bdbe-1508d81244d4)  
   Masuk ke panel admin, cari opsi untuk menghapus pengguna, dan hapus pengguna dengan nama "carlos."

---

**Penjelasan mengapa `Algorithm` diubah:**  
Bagian `Algorithm` dalam header JWT menentukan metode yang digunakan untuk memverifikasi tanda tangan token. Dengan mengubahnya dari `RS256` (yang membutuhkan kunci publik dan privat untuk validasi) menjadi `none`, server tidak lagi memerlukan tanda tangan untuk memvalidasi token. Jika server tidak dikonfigurasi dengan benar, ini memungkinkan penyerang memalsukan JWT dengan payload yang dimodifikasi tanpa tanda tangan valid, sehingga mem-bypass proses autentikasi. Dalam kasus ini, nilai `none` memungkinkan manipulasi token untuk mendapatkan akses sebagai administrator. 

