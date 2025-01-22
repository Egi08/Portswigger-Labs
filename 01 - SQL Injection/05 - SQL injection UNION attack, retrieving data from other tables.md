### Serangan SQL Injection UNION: Mengambil Data dari Tabel Lain

Lab ini mengilustrasikan kerentanannya pada filter kategori produk yang memungkinkan serangan SQL injection UNION untuk mengakses data dari tabel lain. Dalam kasus ini, kita akan menggunakan serangan SQL injection UNION untuk menarik data dari tabel `users`, yang berisi kolom `username` dan `password`. Dengan menggabungkan teknik yang telah dipelajari sebelumnya, kita dapat mengeksploitasi kerentanannya untuk memperoleh nama pengguna dan kata sandi, kemudian login sebagai pengguna administrator.

#### 1. **Memahami Query Asli**  
   Pada aplikasi ini, produk ditampilkan berdasarkan kategori yang dipilih. Query SQL yang digunakan untuk menampilkan produk kemungkinan berbentuk seperti ini:

   ```sql
   SELECT description, content FROM posts WHERE category = 'Gifts'
   ```

   Query ini mengembalikan dua kolom: `description` (deskripsi) dan `content` (konten) dari postingan produk.

#### 2. **Menentukan Jumlah Kolom yang Dikembalikan**  
   Untuk mengidentifikasi berapa banyak kolom yang dikembalikan oleh query, kita akan menggunakan serangan SQL injection UNION. Dari percobaan sebelumnya, kita menemukan bahwa query ini mengembalikan dua kolom.

   **Payload yang digunakan untuk memverifikasi jumlah kolom:**
   ```
   /filter?category=Gifts'+union+all+select+NULL,NULL--
   ```

   Dengan menggunakan payload ini, kita bisa memverifikasi bahwa dua kolom dikembalikan dengan menambahkan `NULL` sebagai pengganti data dalam kolom tersebut.

   **Hasil:**  
   Query mengembalikan dua kolom yang kosong, yang berarti query mengembalikan dua kolom.

#### 3. **Mengakses Data dari Tabel Lain dengan UNION Attack**  
   Setelah mengetahui bahwa query mengembalikan dua kolom, kita bisa melanjutkan untuk mengakses data dari tabel lain, dalam hal ini tabel `users`, yang berisi kolom `username` dan `password`. Dengan menggunakan teknik UNION attack, kita bisa menggabungkan data dari tabel `users` ke dalam hasil query yang sudah ada.

   **Payload yang digunakan untuk mengakses data dari tabel `users`:**
   ```
   /filter?category=Gifts'+union+all+select+username,password+from+users--
   ```

   **Penjelasan Payload:**
   - `'+union+all+select+username,password+from+users--` adalah payload yang menyuntikkan hasil dari query yang mengakses kolom `username` dan `password` dari tabel `users`.
   - `--` adalah komentar dalam SQL yang mengabaikan sisa query yang ada setelahnya.

   **Hasil:**  
   Query akan mengembalikan data dari tabel `users`, seperti nama pengguna dan kata sandi.

   ![Gambar 1](images/SQL%20injection%20UNION%20attack,%20retrieving%20data%20from%20other%20tables/3.png)

#### 4. **Login sebagai Administrator**  
   Setelah memperoleh nama pengguna dan kata sandi dari tabel `users`, kita dapat menggunakannya untuk login sebagai pengguna administrator dan mengakses aplikasi dengan hak akses penuh.

#### 5. **Kesimpulan**  
   Dengan melakukan serangan SQL injection UNION yang mengakses data dari tabel lain, kita dapat menarik informasi penting seperti nama pengguna dan kata sandi yang disimpan dalam tabel `users`. Teknik ini dapat digunakan untuk mengeksploitasi aplikasi yang memiliki kerentanan SQL injection, terutama jika aplikasi tidak memvalidasi input dengan benar dan memungkinkan penggunaan UNION attack.

### Referensi:
- [SQL Injection UNION Attack - PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks)
