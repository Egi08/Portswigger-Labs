
# Reflected DOM XSS

#### Deskripsi Kerentanan

Lab ini menunjukkan **kerentanan Cross-Site Scripting (XSS) berbasis DOM yang direfleksikan**. Kerentanan ini terjadi ketika aplikasi sisi server memproses data dari permintaan dan mengembalikannya dalam respons. Skrip di halaman kemudian memproses data yang direfleksikan tersebut dengan cara yang tidak aman, akhirnya menuliskannya ke dalam **sink** yang berbahaya.

Untuk menyelesaikan lab ini, Anda perlu membuat injeksi yang memanggil fungsi `alert()`.

---------------------------------------------

#### Referensi:

- [PortSwigger: DOM-Based Cross-Site Scripting](https://portswigger.net/web-security/cross-site-scripting/dom-based)

---------------------------------------------

#### Langkah-Langkah Proof of Concept (PoC)

##### 1. Memahami Fungsi Pencarian yang Rentan

Aplikasi ini memiliki fungsi pencarian yang memungkinkan pengguna memasukkan istilah pencarian. Namun, fungsi ini rentan terhadap serangan XSS berbasis DOM karena tidak memproses input pengguna dengan benar.

**Kode JavaScript di `/resources/js/searchResults.js`:**

```javascript
function search(path) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", path + window.location.search);
    xhr.send();

    function displaySearchResults(searchResultsObj) {
        var blogHeader = document.getElementsByClassName("blog-header")[0];
        var blogList = document.getElementsByClassName("blog-list")[0];
        var searchTerm = searchResultsObj.searchTerm
        var searchResults = searchResultsObj.results

        var h1 = document.createElement("h1");
        h1.innerText = searchResults.length + " search results for '" + searchTerm + "'";
        blogHeader.appendChild(h1);
        var hr = document.createElement("hr");
        blogHeader.appendChild(hr)

        for (var i = 0; i < searchResults.length; ++i)
        {
            var searchResult = searchResults[i];
            if (searchResult.id) {
                var blogLink = document.createElement("a");
                blogLink.setAttribute("href", "/post?postId=" + searchResult.id);

                if (searchResult.headerImage) {
                    var headerImage = document.createElement("img");
                    headerImage.setAttribute("src", "/image/" + searchResult.headerImage);
                    blogLink.appendChild(headerImage);
                }

                blogList.appendChild(blogLink);
            }

            blogList.innerHTML += "<br/>";

            if (searchResult.title) {
                var title = document.createElement("h2");
                title.innerText = searchResult.title;
                blogList.appendChild(title);
            }

            if (searchResult.summary) {
                var summary = document.createElement("p");
                summary.innerText = searchResult.summary;
                blogList.appendChild(summary);
            }

            if (searchResult.id) {
                var viewPostButton = document.createElement("a");
                viewPostButton.setAttribute("class", "button is-small");
                viewPostButton.setAttribute("href", "/post?postId=" + searchResult.id);
                viewPostButton.innerText = "View post";
            }
        }

        var linkback = document.createElement("div");
        linkback.setAttribute("class", "is-linkback");
        var backToBlog = document.createElement("a");
        backToBlog.setAttribute("href", "/");
        backToBlog.innerText = "Back to Blog";
        linkback.appendChild(backToBlog);
        blogList.appendChild(linkback);
    }
}
```

##### 2. Identifikasi Sink yang Rentan

Sink adalah titik dalam kode di mana data yang tidak dipercaya (input pengguna) digunakan secara langsung, memungkinkan eksekusi kode berbahaya.

**Gambar Sink yang Rentan:**

![Sink yang Rentan](images/Reflected%20DOM%20XSS/1.png)

##### 3. Memahami Permintaan ke `/search-results`

Aplikasi melakukan permintaan ke endpoint `/search-results` dengan parameter pencarian. Respons dari endpoint ini akan diproses oleh fungsi JavaScript yang rentan.

**Permintaan ke `/search-results`:**

![Permintaan ke /search-results](images/Reflected%20DOM%20XSS/2.png)

##### 4. Respons dari `/search-results`

Respons dari permintaan ini berupa JSON yang kemudian dievaluasi menggunakan `eval()`. Namun, karena tidak ada sanitasi, ini memungkinkan injeksi kode berbahaya.

**Contoh Respons:**

![Respons dari /search-results](images/Reflected%20DOM%20XSS/3.png)

##### 5. Membuat Payload XSS

Untuk menyisipkan dan mengeksekusi skrip berbahaya, kita perlu memodifikasi input pencarian sehingga bagian dari respons JSON diubah untuk menyertakan kode berbahaya.

**Payload XSS yang Benar dari Solusi:**

```
\"-alert(1)}//
```

**Penjelasan Payload:**

- `\"` : Mengakhiri string yang ada dengan menambahkan backslash sebelum tanda kutip ganda.
- `-alert(1)` : Menutup objek JSON dan memanggil fungsi `alert(1)`.
- `}//` : Menutup blok JavaScript dan menambahkan komentar untuk mengabaikan sisa kode.

**Gambar Payload XSS:**

![Payload XSS](images/Reflected%20DOM%20XSS/4.png)

##### 6. Langkah-Langkah Eksploitasi

1. **Masukkan Payload ke dalam Fungsi Pencarian:**

   Pada kotak pencarian, masukkan payload berikut:

   ```
   \"-alert(1)}//
   ```

2. **Kirimkan Permintaan Pencarian:**

   Ketika permintaan dikirimkan, aplikasi akan mengambil respons dari `/search-results` yang berisi payload berbahaya.

3. **Amati Eksekusi Kode:**

   Respons JSON yang dimodifikasi akan dievaluasi oleh `eval()`, menjalankan `alert(1)` di browser korban.

   **Contoh Eksekusi:**

   ```
   <h1>-alert(1)}// search results for '\"-alert(1)}//'</h1>
   ```

   Popup alert akan muncul di browser:

   ![Eksekusi Payload](images/Reflected%20DOM%20XSS/4.png)

##### 7. Alternatif Payload dari Solusi Resmi

Solusi resmi dari lab ini menggunakan payload serupa untuk memicu alert:

```
{{$on.constructor('alert(1)')()}}
```

Payload ini juga akan memicu popup alert yang sama ketika dieksekusi.

##### 8. Penjelasan Query dan Payload

- **Ekspresi JSON yang Rentan:**

  Respons JSON yang tidak disanitasi dan dievaluasi menggunakan `eval()` memungkinkan penyerang menyisipkan kode JavaScript berbahaya.

- **Payload XSS:**

  ```
  \"-alert(1)}//
  ```

  - **`\"`**: Mengakhiri string JSON yang ada.
  - **`-alert(1)`**: Memanggil fungsi `alert(1)`.
  - **`}//`**: Menutup objek JSON dan menambahkan komentar untuk mengabaikan sisa kode.

  Dengan payload ini, ketika respons JSON dievaluasi, JavaScript berbahaya akan dijalankan, memungkinkan penyerang untuk melakukan tindakan seperti mencuri data, mengubah tampilan situs, atau menjalankan skrip lebih lanjut.

#### Pencegahan Kerentanan XSS

Untuk mencegah kerentanan XSS seperti ini dalam aplikasi Anda, beberapa langkah berikut dapat diambil:

1. **Hindari Penggunaan `eval()`:**
   - Fungsi `eval()` dapat menjalankan kode JavaScript dinamis dan harus dihindari kecuali benar-benar diperlukan. Sebagai gantinya, gunakan metode parsing JSON yang aman seperti `JSON.parse()`.

2. **Sanitasi dan Validasi Input:**
   - Selalu bersihkan dan validasi input pengguna sebelum diproses atau ditampilkan. Gunakan pustaka sanitasi yang terpercaya untuk menghapus atau mengkodekan karakter berbahaya.

3. **Gunakan `innerText` atau `textContent` Daripada `innerHTML`:**
   - Jika memungkinkan, gunakan `innerText` atau `textContent` untuk menambahkan teks ke elemen DOM daripada `innerHTML`, yang bisa dieksploitasi untuk menyisipkan HTML atau JavaScript berbahaya.

4. **Implementasikan Content Security Policy (CSP):**
   - CSP dapat membantu mengurangi dampak serangan XSS dengan membatasi sumber konten yang diizinkan untuk dijalankan di browser.

5. **Audit dan Uji Keamanan Secara Berkala:**
   - Lakukan audit kode dan pengujian penetrasi secara rutin untuk mengidentifikasi dan memperbaiki kerentanan sebelum dapat dimanfaatkan oleh penyerang.

6. **Gunakan Framework yang Aman:**
   - Banyak framework modern memiliki mekanisme built-in untuk mencegah XSS. Pastikan untuk memanfaatkan fitur-fitur ini dan tetap memperbarui framework Anda ke versi terbaru.

Dengan menerapkan langkah-langkah di atas, Anda dapat secara signifikan mengurangi risiko serangan XSS dan meningkatkan keamanan aplikasi web Anda.

### Kesimpulan

Reflected DOM XSS adalah salah satu jenis kerentanan yang sering ditemui dalam aplikasi web. Dengan memahami bagaimana kerentanan ini bekerja dan bagaimana cara mengeksploitasinya, pengembang dapat lebih waspada dan menerapkan langkah-langkah pencegahan yang efektif. Selalu prioritaskan keamanan dalam pengembangan aplikasi untuk melindungi data pengguna dan integritas sistem Anda.
