# 🔐 Write-Up: Crackme4

> **Platform:** crackmes.one  
> **Kategori:** Reverse Engineering  
> **Tingkat Kesulitan:** Easy / Beginner  
> **Tools:** Ghidra

---

# 📖 Pendahuluan

Pada write-up ini saya akan menganalisis challenge **Crackme4** dari **crackmes.one** menggunakan **Ghidra**.

Tujuan challenge ini adalah menemukan **login** dan **password** yang benar agar program menampilkan pesan keberhasilan.

Seluruh analisis dilakukan menggunakan **Static Analysis**, tanpa melakukan brute force maupun patching terhadap binary.

---

# 🔍 1. Identifikasi File

Langkah pertama adalah mengidentifikasi informasi dasar dari binary menggunakan utilitas `file`.

```bash
file crackme4.exe
```

Output:

```text
crackme4.exe: PE32+ executable for MS Windows 6.00 (console), x86-64, 6 sections
```

## Analisis

Berdasarkan output tersebut diperoleh informasi sebagai berikut.

| Informasi | Nilai |
|-----------|--------|
| Format | PE32+ |
| Sistem Operasi | Microsoft Windows |
| Arsitektur | x86-64 (64-bit) |
| Jenis Aplikasi | Console Application |
| Jumlah Section | 6 |

Binary ini merupakan executable Windows 64-bit berbentuk aplikasi console sehingga interaksi dilakukan melalui terminal menggunakan input dan output standar.

> **Screenshot**

![File Information](images/file-information.png)

---

# 🔍 2. Reconnaissance

Setelah mengetahui format binary, langkah berikutnya adalah melihat string yang terdapat di dalam executable menggunakan fitur **Defined Strings** pada Ghidra.

Ditemukan beberapa string yang menarik.

```text
Welcome to my so eazy crack me good luck by whekkees

Please enter ur login:

Please enter ur password:

Nice job bro
```

String-string tersebut menunjukkan bahwa program memiliki mekanisme autentikasi berupa **login** dan **password**.

Selain itu, belum ditemukan indikasi adanya proses enkripsi ataupun algoritma yang kompleks.

> **Screenshot**

![Defined Strings](images/defined-strings.png)

---

# 🔍 3. Menelusuri Cross References (XREF)

Selanjutnya saya memilih string:

```text
Please enter ur password:
```

Kemudian melihat **Cross References (XREF)**.

Dari hasil XREF terlihat bahwa string tersebut hanya dipanggil pada fungsi:

```text
FUN_1400012a0
```

Karena seluruh proses autentikasi berada pada fungsi tersebut, maka analisis difokuskan pada fungsi ini.

> **Screenshot**

![Cross Reference](images/xref-password.png)

---

# 🧠 4. Analisis Fungsi `FUN_1400012a0`

Decompiler Ghidra memperlihatkan bahwa fungsi ini menangani seluruh proses login.

Di bagian awal fungsi ditemukan dua pemanggilan berikut.

```cpp
builtin_strncpy(local_70,"whekkes",8);

builtin_strncpy(local_78,"qwerty",7);
```

Fungsi `builtin_strncpy()` digunakan untuk menyalin string konstan ke dalam variabel lokal.

Artinya program telah menyimpan credential secara langsung di dalam binary.

---

## Validasi Login

Program terlebih dahulu meminta login.

```text
Please enter ur login:
```

Input tersebut kemudian dibandingkan menggunakan fungsi:

```cpp
memcmp(...)
```

dengan isi variabel:

```cpp
local_70
```

yang sebelumnya telah diisi menggunakan:

```cpp
builtin_strncpy(local_70,"whekkes",8);
```

Sehingga login yang benar adalah:

```text
whekkes
```

---

## Validasi Password

Apabila login sesuai, program meminta password.

```text
Please enter ur password:
```

Password kemudian dibandingkan menggunakan:

```cpp
memcmp(...)
```

dengan isi variabel:

```cpp
local_78
```

yang sebelumnya telah diisi menggunakan:

```cpp
builtin_strncpy(local_78,"qwerty",7);
```

Sehingga password yang benar adalah:

```text
qwerty
```

Dengan demikian, challenge ini menggunakan mekanisme **hardcoded credential**, yaitu nilai login dan password telah disimpan langsung di dalam executable.

> **Screenshot**

![Function Analysis](images/function-analysis.png)

---

# 🚀 5. Solusi

Berdasarkan hasil analisis diperoleh credential berikut.

| Input | Nilai |
|--------|-------|
| Login | `whekkes` |
| Password | `qwerty` |

Tidak diperlukan proses brute force maupun analisis algoritma yang rumit karena credential sudah tersimpan secara langsung pada binary.

---

# ✅ 6. Verifikasi

Program kemudian dijalankan menggunakan credential yang telah ditemukan.

```text
Please enter ur login:
whekkes

Please enter ur password:
qwerty

Nice job bro
```

Program berhasil menerima login dan password sehingga menampilkan pesan:

```text
Nice job bro
```

Hal ini membuktikan bahwa hasil analisis telah sesuai.

> **Screenshot**

![Solved](images/solved.png)

---

# 🎯 Hasil

| Item | Nilai |
|------|-------|
| Challenge | Crackme4 |
| Platform | crackmes.one |
| Metode | Static Analysis |
| Tools | Ghidra |
| Login | `whekkes` |
| Password | `qwerty` |
| Status | ✅ Solved |

---

# 💡 Hal yang Dipelajari

Melalui challenge ini saya mempelajari beberapa konsep dasar Reverse Engineering, antara lain:

- Mengidentifikasi format executable menggunakan utilitas `file`.
- Melakukan reconnaissance melalui **Defined Strings**.
- Menggunakan **Cross References (XREF)** untuk menemukan fungsi yang menggunakan suatu string.
- Membaca pseudocode hasil dekompilasi Ghidra.
- Memahami fungsi `builtin_strncpy()` sebagai proses penyalinan string konstan.
- Memahami penggunaan `memcmp()` untuk membandingkan input pengguna dengan credential yang telah disimpan.

---

# 📝 Kesimpulan

Challenge **Crackme4** merupakan contoh sederhana mekanisme autentikasi yang menggunakan **hardcoded credential**.

Dengan memanfaatkan utilitas `file`, fitur **Defined Strings**, **Cross References**, dan **Decompiler** pada Ghidra, login serta password dapat ditemukan tanpa perlu menjalankan debugger ataupun melakukan brute force.

Challenge ini memberikan pemahaman mengenai alur analisis statis, mulai dari identifikasi binary hingga memahami logika autentikasi melalui pembacaan pseudocode.
