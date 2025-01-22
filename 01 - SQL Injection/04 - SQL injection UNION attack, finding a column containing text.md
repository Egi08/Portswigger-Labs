### Penjelasan Serangan SQL Injection UNION: Menemukan Kolom yang Mengandung Teks

Lab ini memperlihatkan kerentanan SQL injection dalam filter kategori produk. Dengan serangan SQL injection UNION, kita dapat mengakses data dari tabel lain. Setelah menentukan jumlah kolom yang dikembalikan oleh query, langkah selanjutnya adalah mengidentifikasi kolom yang kompatibel dengan data bertipe string.

Lab ini akan memberikan nilai acak yang perlu ditampilkan dalam hasil query. Untuk menyelesaikan lab ini, kita akan melakukan serangan SQL injection UNION yang mengembalikan baris tambahan yang berisi nilai yang diberikan. Teknik ini akan membantu kita menentukan kolom mana yang kompatibel dengan data bertipe string.

#### 1. **Memahami Query Asli**  
   Pada aplikasi ini, produk ditampilkan berdasarkan kategori yang dipilih. Query SQL yang digunakan untuk menampilkan produk kemungkinan besar berbentuk seperti ini:

   ```sql
   SELECT name, price FROM products WHERE category = 'Accessories'
   ```

   Query ini mengembalikan dua kolom: `name` (nama produk) dan `price` (harga produk).

#### 2. **Memverifikasi Kerentanannya dan Menentukan Jumlah Kolom**  
   Langkah pertama adalah melakukan serangan SQL injection UNION untuk memastikan jumlah kolom yang dikembalikan oleh query. Berdasarkan serangan sebelumnya, kita sudah mengetahui bahwa query ini mengembalikan tiga kolom.

   **Payload yang digunakan untuk mengeksploitasi kerentanannya:**
   ```
   /filter?category=Accessories'+union+all+select+NULL,NULL,NULL--
   ```

   Payload ini menggunakan `NULL` untuk mengidentifikasi bahwa query mengembalikan tiga kolom. Hasilnya menunjukkan tiga kolom yang kosong atau tidak terisi.

#### 3. **Langkah Kedua: Menemukan Kolom yang Kompatibel dengan Data Teks**  
   Setelah mengetahui bahwa query mengembalikan tiga kolom, langkah berikutnya adalah mencari kolom yang kompatibel dengan data teks. Dalam hal ini, kita ingin menampilkan string acak yang diberikan, misalnya "Qrc0Pq". Untuk melakukan ini, kita perlu memasukkan string tersebut pada kolom yang tepat.

   **Payload yang digunakan untuk menyuntikkan string "Qrc0Pq":**
   ```
   /filter?category=Accessories'+union+all+select+'0','Qrc0Pq','1234'--
   ```

   Di sini, kita mengganti kolom kedua dengan string "Qrc0Pq" dan kolom lainnya dengan nilai lain (dalam contoh ini, '0' dan '1234'). Jika kolom kedua menerima data teks, maka kita akan melihat nilai "Qrc0Pq" ditampilkan dalam hasil query.

   **Hasil:**  
   Setelah mengirimkan payload ini, kita dapat melihat string "Qrc0Pq" muncul di hasil query pada kolom kedua.

   ![Gambar 1](images/SQL%20injection%20UNION%20attack,%20finding%20a%20column%20containing%20text/2.png)

#### 4. **Kesimpulan**  
   Dengan menggunakan serangan SQL injection UNION, kita berhasil menentukan kolom mana yang kompatibel dengan data teks. Teknik ini penting untuk mengidentifikasi kolom yang dapat kita manfaatkan untuk menarik data lebih lanjut dari tabel yang lebih sensitif, atau untuk menginjeksi nilai tertentu dalam hasil query.

### Referensi:
- [SQL Injection UNION Attack - PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks)
