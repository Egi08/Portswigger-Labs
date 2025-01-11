
# DOM XSS in AngularJS expression with angle brackets and double quotes HTML-encoded

#### Deskripsi Kerentanan

Lab ini mengandung **kerentanan Cross-Site Scripting (XSS) berbasis DOM** pada ekspresi AngularJS dalam fungsi pencarian. AngularJS adalah pustaka JavaScript populer yang memindai konten node HTML yang mengandung atribut `ng-app` (juga dikenal sebagai direktif AngularJS). Ketika direktif ditambahkan ke kode HTML, Anda dapat menjalankan ekspresi JavaScript di dalam tanda kurung kurawal ganda (`{{ }}`). Teknik ini berguna ketika kurung sudut (`<` dan `>`) dikodekan.

Untuk menyelesaikan lab ini, lakukan serangan Cross-Site Scripting yang menjalankan ekspresi AngularJS dan memanggil fungsi `alert`.

---------------------------------------------

#### Referensi:

- [PortSwigger: DOM-Based Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/dom-based)

---------------------------------------------

#### Langkah-Langkah PoC

1. **Fungsi Pencarian yang Rentan:**

    Terdapat fungsi pencarian di aplikasi ini yang memungkinkan pengguna memasukkan string pencarian. Namun, fungsi ini memiliki kerentanan XSS berbasis DOM karena tidak memproses input pengguna dengan benar.

    ![Fungsi Pencarian](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/1.png)

2. **String sebagai Bagian dari Tag `<h1>`:**

    Hasil pencarian ditampilkan dalam tag `<h1>`. Namun, karakter khusus seperti `>`, `<`, dan `'` dikodekan menjadi entitas HTML (`&gt;`, `&lt;`, dan `&#39;`).

    ![Tag h1 dengan String](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/2.png)

3. **Pencarian Karakter Khusus:**

    Mencari string seperti `"<'>"` menunjukkan bahwa karakter `>`, `<`, dan `'` telah dikodekan.

    ![Karakter Khusus yang Dikodekan](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/3.png)

4. **Penggunaan Kurung Kurawal Ganda:**

    Menggunakan kurung kurawal ganda (`{{ }}`) memungkinkan eksekusi ekspresi AngularJS. Misalnya:

    ```
    {{1== 1 ? "Yes, it is equal" : "No, it is not"}}
    ```

    Ekspresi ini akan dievaluasi oleh AngularJS dan menampilkan hasilnya.

    ![Ekspresi AngularJS](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/4.png)
    ![Hasil Ekspresi](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/5.png)

5. **Menyisipkan Payload XSS:**

    Dengan memahami bahwa AngularJS akan mengevaluasi ekspresi di dalam kurung kurawal ganda, kita dapat menyisipkan payload yang berbahaya untuk menjalankan kode JavaScript. Misalnya, untuk memunculkan alert, kita dapat menggunakan konstruktor `constructor` dalam AngularJS:

    ```
    {{constructor.constructor('alert(1)')()}}
    ```

    Payload ini memanfaatkan kemampuan JavaScript untuk mengakses konstruktor `Function` melalui `constructor`, yang kemudian menjalankan fungsi `alert(1)`.

    ![Payload XSS](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/6.png)

    **Catatan:** Menggunakan `constructor.constructor` memungkinkan eksekusi kode arbitrer karena dapat memanggil fungsi `Function` secara dinamis. Alternatif lain yang sering digunakan adalah:

    ```
    {{$on.constructor('alert(1)')()}}
    ```

    Ini juga berfungsi dengan cara yang sama untuk memicu alert.

#### Penjelasan Query dan Payload

- **Ekspresi AngularJS:**

    ```
    {{1== 1 ? "Yes, it is equal" : "No, it is not"}}
    ```

    Ekspresi ini adalah contoh sederhana yang menampilkan "Yes, it is equal" jika kondisi benar. Namun, karena AngularJS mengevaluasi ekspresi di dalam kurung kurawal ganda, kita dapat menyisipkan ekspresi yang lebih kompleks dan berbahaya.

- **Payload XSS:**

    ```
    {{constructor.constructor('alert(1)')()}}
    ```

    - `constructor.constructor` mengakses konstruktor `Function` dari objek JavaScript, memungkinkan eksekusi kode dinamis.
    - `'alert(1)'` adalah kode yang ingin dijalankan, dalam hal ini memunculkan popup alert dengan pesan "1".
    - `()` di akhir ekspresi memanggil fungsi yang telah dibuat.

    Dengan payload ini, ketika pengguna mengunjungi URL yang berisi ekspresi tersebut, JavaScript berbahaya akan dieksekusi di browser mereka, memungkinkan penyerang untuk melakukan tindakan berbahaya seperti mencuri data, mengubah tampilan situs, atau menjalankan skrip lebih lanjut.

#### Langkah-Langkah Eksploitasi

1. **Memasukkan Payload ke Fungsi Pencarian:**

    Masukkan payload XSS ke dalam kotak pencarian:

    ```
    {{constructor.constructor('alert(1)')()}}
    ```

2. **Mengamati Eksekusi Kode:**

    Setelah memasukkan payload dan mengirimkan pencarian, aplikasi akan menampilkan hasil dalam tag `<h1>`. Karena AngularJS mengevaluasi ekspresi tersebut, popup alert akan muncul di browser korban:

    ```
    <h1>{{constructor.constructor('alert(1)')()}}</h1>
    ```

    ![Eksekusi Payload](images/DOM%20XSS%20in%20AngularJS%20expression%20with%20angle%20brackets%20and%20double%20quotes%20HTML-encoded/6.png)

3. **Menggunakan Solusi Resmi:**

    Solusi resmi dari lab ini menggunakan payload serupa:

    ```
    {{$on.constructor('alert(1)')()}}
    ```

    Ini juga akan memicu alert yang sama ketika dieksekusi.

#### Pencegahan Kerentanan XSS pada AngularJS

Untuk mencegah kerentanan XSS seperti ini dalam aplikasi AngularJS, beberapa langkah yang dapat diambil meliputi:

1. **Sanitasi Input Pengguna:**
    - Selalu bersihkan dan validasi input pengguna sebelum diproses atau ditampilkan di halaman web.
    - Gunakan pustaka sanitasi yang terpercaya untuk menghapus atau mengkodekan karakter berbahaya.

2. **Gunakan `ngSanitize`:**
    - AngularJS menyediakan modul `ngSanitize` yang dapat membantu mencegah injeksi kode berbahaya dengan membersihkan HTML yang tidak aman.

3. **Batasi Penggunaan Ekspresi AngularJS:**
    - Hindari menggunakan ekspresi AngularJS di area yang menerima input pengguna tanpa validasi yang tepat.
    - Pertimbangkan untuk menggunakan metode rendering yang lebih aman seperti AngularJS's `ng-bind` daripada interpolasi langsung dengan `{{ }}`.

4. **Implementasikan Content Security Policy (CSP):**
    - CSP dapat membantu mengurangi dampak serangan XSS dengan membatasi sumber konten yang diizinkan untuk dijalankan di browser.

5. **Audit dan Uji Keamanan Secara Berkala:**
    - Lakukan audit kode dan pengujian penetrasi secara rutin untuk mengidentifikasi dan memperbaiki kerentanan sebelum dimanfaatkan oleh penyerang.

Dengan mengikuti langkah-langkah di atas, Anda dapat mengurangi risiko serangan XSS dan meningkatkan keamanan aplikasi web Anda.
