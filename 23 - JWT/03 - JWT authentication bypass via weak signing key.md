Berikut adalah versi terjemahan dan penjelasan dalam Bahasa Indonesia:

---

# JWT authentication bypass melalui kunci penandatanganan yang lemah

Lab ini menggunakan mekanisme berbasis JWT untuk menangani sesi. Lab ini menggunakan kunci rahasia yang sangat lemah untuk menandatangani dan memverifikasi token. Kunci ini dapat dengan mudah diretas menggunakan daftar kata (wordlist) dari kunci rahasia umum.

Untuk menyelesaikan lab ini, pertama-tama brute-force kunci rahasia situs web. Setelah mendapatkan kunci tersebut, gunakan kunci itu untuk menandatangani token sesi yang dimodifikasi agar memberikan akses ke panel admin di /admin, lalu hapus pengguna carlos.

Anda dapat masuk ke akun Anda sendiri menggunakan kredensial berikut: wiener:peter

**Tips:** Kami merekomendasikan untuk membiasakan diri dengan cara kerja JWT di Burp Suite sebelum mencoba lab ini.

Kami juga merekomendasikan menggunakan hashcat untuk brute-force kunci rahasia. Untuk detail tentang cara melakukannya, lihat [Brute forcing secret keys using hashcat](https://portswigger.net/web-security/jwt).

---------------------------------------------

**Referensi:**  
- [JWT Authentication Bypass](https://portswigger.net/web-security/jwt)  
- [JWT Secrets List](https://raw.githubusercontent.com/wallarm/jwt-secrets/master/jwt.secrets.list)

---------------------------------------------

### Bukti Konsep (Proof of Concept - POC)

1. **Login dengan akun wiener:peter**  
   ![image](https://github.com/user-attachments/assets/b3e4811a-e5fb-4212-bbc3-bf798822e454)  
   Masuk ke aplikasi menggunakan kredensial yang diberikan untuk mendapatkan token sesi awal.

2. **Buka OWASP Penetration Testing Kit**  
   ![image](https://github.com/user-attachments/assets/1a631c81-ef99-4c49-9a7d-f12454db6ec4)  
   Gunakan OWASP Penetration Testing Kit atau alat seperti Burp Suite untuk menganalisis dan memodifikasi token JWT.

3. **Pilih opsi Attack -> Secret brute-force**  
   ![image](https://github.com/user-attachments/assets/15147e02-d2fc-4d3c-ad78-acd155a0a8c3)  
   Gunakan fitur brute-force untuk menemukan kunci rahasia JWT.

4. **Klik brute-force, salin kunci rahasia yang ditemukan (contoh: "the secret is: [kunci_terdeteksi]")**  
   ![image](https://github.com/user-attachments/assets/69345ebb-9140-4230-9c45-11db8f526eea)  
   Setelah proses brute-force selesai, catat kunci rahasia yang ditemukan.

5. **Ubah nilai `sub` menjadi `administrator`, masukkan kunci rahasia tadi ke bagian signature, lalu perbarui di semua sumber**  
   ![image](https://github.com/user-attachments/assets/6cad9bd5-3f84-4858-b607-3ce7aee4fe5c)  
   Gunakan kunci rahasia untuk menandatangani ulang token JWT dengan nilai payload yang dimodifikasi agar memiliki akses sebagai administrator.

**Penjelasan:**  
Mengubah nilai `sub` menjadi `administrator` memungkinkan kita untuk mendapatkan hak akses admin. Signature JWT menggunakan kunci rahasia untuk menjamin integritas token. Karena kunci rahasia yang digunakan oleh server sangat lemah, penyerang dapat menemukannya dengan brute-force, kemudian menandatangani ulang token yang telah dimodifikasi, membuat token tersebut dianggap valid oleh server.

6. **Muat ulang web untuk menampilkan panel admin**  
   ![image](https://github.com/user-attachments/assets/de8c14dd-5605-49bf-8407-65e4a1c5b29d)  
   Setelah token diperbarui, muat ulang halaman. Panel admin sekarang dapat diakses.

7. **Hapus pengguna carlos**  
   ![image](https://github.com/user-attachments/assets/84de6f50-58ee-40c4-875a-a2eb10cdcfeb)  
   Masuk ke panel admin, cari opsi untuk menghapus pengguna, dan hapus pengguna dengan nama "carlos."

---

**Penjelasan mengapa kunci rahasia ditemukan dan digunakan:**  
JWT menggunakan kunci rahasia untuk menandatangani dan memverifikasi token. Jika kunci rahasia lemah atau umum, penyerang dapat menemukannya melalui brute-force menggunakan daftar kata atau alat seperti hashcat. Setelah kunci rahasia ditemukan, token JWT dapat dimodifikasi dan ditandatangani ulang sehingga dianggap valid oleh server, meskipun token tersebut telah dimanipulasi. Ini adalah celah keamanan serius yang dapat dimanfaatkan untuk mendapatkan akses tidak sah.
