### Penjelasan Mengenai Serangan SQL Injection UNION: Menentukan Jumlah Kolom yang Dikembalikan oleh Query

Lab ini mengilustrasikan kerentanan SQL injection dalam filter kategori produk. Dengan menggunakan serangan SQL injection UNION, kita dapat mengakses data dari tabel lain. Langkah pertama dalam serangan UNION adalah menentukan jumlah kolom yang dikembalikan oleh query. Setelah itu, kita dapat menggunakan teknik ini dalam lab-lab berikutnya untuk membangun serangan penuh.

Berikut adalah penjelasan tentang bagaimana melakukan serangan SQL injection UNION untuk menentukan jumlah kolom yang dikembalikan oleh query:

#### 1. **Memahami Query Asli**  
   Aplikasi ini menampilkan produk berdasarkan kategori yang dipilih oleh pengguna. Query SQL yang digunakan untuk menampilkan produk mungkin terlihat seperti ini:

   ```sql
   SELECT name, price FROM products WHERE category = 'Accessories'
   ```

   Dalam query ini, hanya dua kolom yang dikembalikan: `name` (nama produk) dan `price` (harga produk).

#### 2. **Mendeteksi Kerentanannya**  
   Kerentanannya terdapat pada bagaimana aplikasi memproses input pengguna tanpa melakukan sanitasi yang tepat, memungkinkan kita untuk menyuntikkan perintah SQL tambahan untuk mengubah hasil query. Dalam hal ini, kita akan melakukan serangan UNION untuk memeriksa jumlah kolom yang dikembalikan oleh query.

#### 3. **Langkah Pertama: Mencoba Menyuntikkan Nilai NULL**  
   Untuk mengidentifikasi jumlah kolom yang dikembalikan oleh query, kita mulai dengan menyuntikkan `NULL` pada bagian parameter query. Jika query menerima nilai `NULL` untuk setiap kolom yang dikembalikan, kita bisa mengetahui berapa banyak kolom yang dikembalikan.

   **Payload yang digunakan:**
   ```
   /filter?category=Accessories'+union+select+NULL,NULL,NULL--
   ```

   Dalam payload ini, kita menambahkan tiga nilai `NULL` untuk menggantikan hasil asli query. Ini memberikan petunjuk mengenai jumlah kolom yang dikembalikan. Jika query menampilkan nilai `NULL` untuk setiap kolom, berarti jumlah kolom yang dikembalikan adalah tiga, bukan dua.

   **Hasil:**  
   Setelah mengirimkan payload ini, kita melihat hasil query dengan tiga kolom (dua kolom asli dan satu kolom tambahan yang kita sisipkan dengan `NULL`).

   ![Gambar 1](images/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query/3.png)

#### 4. **Langkah Kedua: Menyuntikkan Nilai Selain NULL**  
   Setelah mengetahui jumlah kolom yang dikembalikan, kita dapat melanjutkan dengan menyuntikkan nilai-nilai lain, seperti angka atau string, untuk mengonfirmasi bahwa query dapat menerima berbagai jenis data.

   **Payload yang digunakan:**
   ```
   /filter?category=Accessories'+union+all+select+'0','1','2'--
   ```

   Dalam payload ini, kita mengganti `NULL` dengan nilai `'0'`, `'1'`, dan `'2'`. Ini untuk memastikan bahwa query dapat menangani berbagai jenis data dan mengonfirmasi bahwa kita benar-benar dapat mengeksploitasi query dengan menambahkan kolom ekstra.

   **Hasil:**  
   Dengan menggunakan payload ini, kita dapat melihat data yang kita masukkan dalam bentuk angka (`'0'`, `'1'`, dan `'2'`), yang menunjukkan bahwa kita telah berhasil mengeksploitasi query dan mengonfirmasi jumlah kolom yang dikembalikan.

   ![Gambar 2](images/SQL%20injection%20UNION%20attack,%20determining%20the%20number%20of%20columns%20returned%20by%20the%20query/4.png)

#### 5. **Kesimpulan**  
   Dengan menggunakan serangan SQL injection UNION ini, kita berhasil menentukan jumlah kolom yang dikembalikan oleh query dan mengeksploitasi kerentanannya untuk mengakses data dari tabel lain. Langkah pertama adalah menyuntikkan `NULL` untuk menentukan jumlah kolom, kemudian menggantinya dengan nilai lain untuk mengonfirmasi hasilnya.

### Referensi:
- [SQL Injection UNION Attack - PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks)
