# JWT authentication bypass melalui tanda tangan yang tidak diverifikasi

Lab ini menggunakan mekanisme berbasis JWT untuk menangani sesi. Karena kelemahan implementasi, server tidak memverifikasi tanda tangan dari JWT apa pun yang diterimanya.

Untuk menyelesaikan lab ini, modifikasi token sesi Anda untuk mendapatkan akses ke panel admin di /admin, lalu hapus pengguna carlos.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: wiener:peter

**Tips:** Kami merekomendasikan untuk membiasakan diri dengan cara kerja JWT di Burp Suite sebelum mencoba lab ini.

---------------------------------------------

**Referensi:**  
- [JWT Authentication Bypass](https://portswigger.net/web-security/jwt)

---------------------------------------------

### Bukti Konsep (Proof of Concept - POC)

1. **Login dengan akun wiener:peter**  
   ![image](https://github.com/user-attachments/assets/b36b4939-9db9-4cb8-814a-d3a9658d87f0)  
   Masuk ke aplikasi menggunakan kredensial yang diberikan untuk mendapatkan token sesi awal.

2. **Buka OWASP Penetration Testing Kit**  
   ![image](https://github.com/user-attachments/assets/1a631c81-ef99-4c49-9a7d-f12454db6ec4)  
   Gunakan OWASP atau alat seperti Burp Suite untuk menganalisis dan memodifikasi token JWT.

3. **Ubah nilai `sub` menjadi administrator dan perbarui di semua sumber**  
   ![image](https://github.com/user-attachments/assets/cd6e57b0-4af7-486f-9cb0-0102811c1ce3)  
   Ubah payload JWT, khususnya bagian `sub`, dari pengguna asli ke `administrator`. Hal ini dilakukan karena nilai `sub` menunjukkan identitas pengguna. Dengan mengubahnya menjadi `administrator`, Anda mendapatkan hak akses sebagai admin.

4. **Muat ulang web untuk menampilkan panel admin**  
   ![image](https://github.com/user-attachments/assets/355dca1c-9e5c-4556-b7bb-d741eaa31d18)  
   Setelah token diperbarui, muat ulang halaman. Panel admin sekarang dapat diakses.

5. **Hapus pengguna carlos**  
   ![image](https://github.com/user-attachments/assets/98726f29-4261-4f30-ad7d-ea41ae328a03)  
   ![image](https://github.com/user-attachments/assets/9e41a5e8-66d7-4df6-bc05-24a44951efbf)  
   Masuk ke panel admin, cari opsi untuk menghapus pengguna, dan hapus pengguna dengan nama "carlos."

---

**Penjelasan mengapa `sub` diubah:**  
Bagian `sub` dalam payload JWT biasanya menunjukkan identitas pengguna. Ketika server tidak memverifikasi tanda tangan JWT, nilai ini dapat diubah untuk mendapatkan hak akses yang lebih tinggi, seperti menjadi administrator. Dengan mengubah `sub` menjadi `administrator`, server akan menganggap Anda sebagai admin tanpa melakukan validasi lebih lanjut. Hal ini menunjukkan celah keamanan serius dalam implementasi JWT di lab ini. 
