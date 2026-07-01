# 🔐 Write-Up: Getting Started Keygen

> **Platform:** crackmes.one  
> **Kategori:** Reverse Engineering  
> **Tingkat Kesulitan:** Easy / Beginner  
> **Tools:** Ghidra

---

# 📖 Pendahuluan

Pada write-up ini saya akan menganalisis challenge **Getting Started Keygen** dari **crackmes.one** menggunakan **Ghidra**.

Tujuan challenge ini adalah menemukan kombinasi **string** dan **angka** yang benar agar program menerima input dan menampilkan pesan keberhasilan.

Seluruh analisis dilakukan menggunakan pendekatan **Static Analysis**, tanpa melakukan brute force maupun patching terhadap binary.

---

# 🔍 1. Reconnaissance

Sebelum membuka binary menggunakan Ghidra, saya melakukan identifikasi awal menggunakan utilitas `strings`.

```bash
strings getting_started_keygen
```

Output yang menarik adalah sebagai berikut.

```text
Enter a string of characters (no spaces):
Bro, what are you trying to do?
Enter correct number (no spaces):
OMG! You did it! :3
Send help pls.
```

Dari output tersebut dapat disimpulkan bahwa:

- Program meminta sebuah string.
- Program meminta sebuah angka.
- Program memiliki dua kondisi validasi:
  - Input salah → `Bro, what are you trying to do?`
  - Input benar → `OMG! You did it! :3`

Output ini memberikan gambaran awal mengenai alur program sebelum dilakukan analisis lebih lanjut.

> **Screenshot**

![Strings](images/strings.png)

---

# 🔍 2. Analisis Statis (Main Function)

Binary kemudian dibuka menggunakan **Ghidra**.

Pada fungsi utama (`FUN_001011f0`), terlihat bahwa program meminta dua input dari pengguna:

- Sebuah string
- Sebuah integer

Selanjutnya program melakukan validasi terhadap kedua input tersebut.

---

## Validasi Panjang String

Ditemukan potongan kode berikut.

```cpp
if (5 < local_70 - 5U) {
    exit(0);
}
```

Karena menggunakan **unsigned integer**, kondisi tersebut memastikan panjang string berada pada rentang tertentu.

### Kesimpulan

String harus memiliki panjang:

- Minimal **5 karakter**
- Maksimal **10 karakter**

Jika syarat tersebut tidak terpenuhi, program akan langsung keluar.

---

## Validasi Angka

Setelah menerima input angka, program memanggil fungsi berikut.

```cpp
iVar1 = FUN_001014b0((long *)local_58);

if (local_7c == iVar1) {
    std::cout << "OMG! You did it! :3";
}
```

### Kesimpulan

Nilai integer yang dimasukkan harus sama dengan nilai yang dikembalikan oleh:

```text
FUN_001014b0()
```

Oleh karena itu, fungsi tersebut menjadi fokus analisis berikutnya.

> **Screenshot**

![Main Function](images/main-function.png)

---

# 🧠 3. Analisis Fungsi `FUN_001014b0`

Berikut hasil dekompilasi menggunakan Ghidra.

```cpp
int FUN_001014b0(long *param_1)
{
    ...

    if (param_1[1] != 0) {

        lVar3 = 0;
        iVar4 = 0;

        do {

            pcVar1 = (char *)(*param_1 + lVar3);
            puVar2 = &DAT_00104020 + lVar3;

            lVar3++;

            iVar4 = iVar4 + ((int)*pcVar1 ^ *puVar2);

        } while (param_1[1] != lVar3);

        return iVar4;
    }

    return 0;
}
```

---

## Cara Kerja Algoritma

Fungsi melakukan proses berikut:

1. Melakukan iterasi sebanyak panjang string.
2. Mengambil karakter input.
3. Mengambil nilai dari `DAT_00104020`.
4. Melakukan operasi XOR.
5. Menjumlahkan seluruh hasil XOR.
6. Mengembalikan hasil akhir.

Secara matematis:

```text
Result =
(input[0] XOR key[0]) +
(input[1] XOR key[1]) +
...
```

> **Screenshot**

![Function Analysis](images/function-analysis.png)

---

# 📦 4. Ekstraksi Data

Pada Listing View Ghidra ditemukan isi array berikut.

| Index | Address | Hex | Decimal |
|------:|---------|-----|--------:|
| 0 | 00104020 | 0x04 | 4 |
| 1 | 00104024 | 0x4F | 79 |
| 2 | 00104028 | 0x81 | 129 |
| 3 | 0010402C | 0xAB | 171 |
| 4 | 00104030 | 0xFE | 254 |

Array tersebut digunakan sebagai nilai pembanding pada operasi XOR.

---

# 🚀 5. Menentukan Solusi

Dipilih string sederhana:

```text
AAAAA
```

Nilai ASCII karakter `A` adalah:

```text
0x41 = 65
```

Perhitungan:

| Karakter | ASCII | Key | XOR | Hasil |
|-----------|------:|----:|----:|------:|
| A | 0x41 | 0x04 | 0x45 | 69 |
| A | 0x41 | 0x4F | 0x0E | 14 |
| A | 0x41 | 0x81 | 0xC0 | 192 |
| A | 0x41 | 0xAB | 0xEA | 234 |
| A | 0x41 | 0xFE | 0xBF | 191 |

Total:

```text
69 + 14 + 192 + 234 + 191 = 700
```

Sehingga didapatkan:

```text
String : AAAAA
Number : 700
```

---

# ✅ 6. Verifikasi

## Input Salah

Percobaan menggunakan:

```text
String : vvhibiwashere
Number : 278
```

Program menampilkan:

```text
Bro, what are you trying to do?
```

Hal ini menunjukkan bahwa angka yang diberikan tidak sesuai dengan hasil perhitungan program.

> **Screenshot**

![Wrong Input](images/wrong-input.png)

---

## Input Benar

Menggunakan hasil analisis:

```text
String : AAAAA
Number : 700
```

Program berhasil menampilkan:

```text
OMG! You did it! :3

Send help pls.
```

Artinya proses validasi berhasil dilewati.

> **Screenshot**

![Correct Input](images/correct-input.png)

---

# 🎯 Hasil

| Item | Nilai |
|------|-------|
| Challenge | Getting Started Keygen |
| Platform | crackmes.one |
| Metode | Static Analysis |
| Tools | Ghidra |
| Status | ✅ Solved |

---

# 💡 Hal yang Dipelajari

Melalui challenge ini saya mempelajari beberapa konsep penting dalam Reverse Engineering, antara lain:

- Melakukan reconnaissance menggunakan `strings`.
- Membaca pseudocode hasil dekompilasi Ghidra.
- Memahami alur logika program.
- Menganalisis fungsi secara statis.
- Mengekstrak data dari memori.
- Memahami operasi XOR.
- Menentukan input yang valid berdasarkan hasil analisis.

---

# 📝 Kesimpulan

Challenge **Getting Started Keygen** merupakan latihan yang baik untuk memahami dasar-dasar Reverse Engineering. Dengan memanfaatkan analisis statis menggunakan Ghidra dan utilitas `strings`, kita dapat mengidentifikasi alur program, memahami algoritma validasi, serta menghitung nilai yang benar tanpa perlu melakukan brute force.

Challenge ini memperkenalkan konsep-konsep penting seperti pembacaan pseudocode, operasi XOR, analisis memori, dan validasi input yang menjadi dasar dalam menganalisis executable yang lebih kompleks di masa mendatang.
