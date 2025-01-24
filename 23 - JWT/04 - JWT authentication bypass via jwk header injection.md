# JWT authentication bypass melalui injeksi header jwk

Lab ini menggunakan mekanisme berbasis JWT untuk menangani sesi. Server mendukung parameter `jwk` dalam header JWT. Ini kadang-kadang digunakan untuk menyematkan kunci verifikasi yang benar langsung dalam token. Namun, server gagal memeriksa apakah kunci yang diberikan berasal dari sumber yang terpercaya.

Untuk menyelesaikan lab ini, modifikasi dan tandatangani JWT yang memberi Anda akses ke panel admin di /admin, lalu hapus pengguna carlos.

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

3. **Ubah nilai `sub` menjadi `administrator` dan pilih opsi Attack -> Jwk injection (CVE-2018-0114)**  
   ![image](https://github.com/user-attachments/assets/02954a61-8793-4cf4-9a4c-6fa03a9e79aa)  
   **Penjelasan:**  
   Ubah nilai `sub` dalam payload JWT menjadi `administrator` untuk memperoleh hak akses sebagai admin. Pilih opsi Jwk injection untuk menyuntikkan kunci yang dihasilkan ke dalam token.

4. **Pilih algoritma 'RS256', buat pasangan kunci, lalu generate dan salin token yang dihasilkan**  
   ![image](https://github.com/user-attachments/assets/28e49e8f-86ad-431c-9453-27d11e4f01ff)  
   Pilih algoritma tanda tangan `RS256`, buat pasangan kunci (public dan private key), lalu gunakan kunci tersebut untuk menghasilkan dan menandatangani ulang token JWT.

5. **Masukkan token tadi pada session0 di cookie, lalu perbarui di semua sumber**  
   ![image](https://github.com/user-attachments/assets/f6aefb3c-9ffb-4c50-a04b-3c0109372d1b)  
   Salin token JWT yang telah dimodifikasi dan masukkan ke dalam cookie browser di bagian session0. Setelah itu, perbarui di semua sumber untuk memastikan token baru digunakan.

6. **Muat ulang web untuk menampilkan panel admin**  
   ![image](https://github.com/user-attachments/assets/de8c14dd-5605-49bf-8407-65e4a1c5b29d)  
   Setelah token diperbarui, muat ulang halaman. Panel admin sekarang dapat diakses.

7. **Hapus pengguna carlos**  
   ![image](https://github.com/user-attachments/assets/df2f1ce3-56fc-4e30-8b19-0d464a839ecd)  
   Masuk ke panel admin, cari opsi untuk menghapus pengguna, dan hapus pengguna dengan nama "carlos."

---

**Penjelasan mengapa header `jwk` diubah:**  
Header `jwk` dalam JWT digunakan untuk menunjukkan lokasi kunci publik yang digunakan untuk memverifikasi tanda tangan JWT. Dalam kasus ini, server gagal memeriksa apakah kunci yang diberikan berasal dari sumber terpercaya, sehingga memungkinkan penyerang untuk menyuntikkan kunci verifikasi mereka sendiri (dalam hal ini, pasangan kunci yang baru dibuat). Dengan mengubah header untuk memasukkan kunci yang kita buat sendiri, kita dapat menandatangani ulang token dengan kunci tersebut, sehingga server menganggap token tersebut valid dan memberikan akses tidak sah ke panel admin. Ini menunjukkan celah keamanan serius dalam cara server memverifikasi tanda tangan JWT.
