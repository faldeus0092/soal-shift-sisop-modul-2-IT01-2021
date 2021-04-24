## Soal Nomor 3
Source code dapat dilihat di sini [soal3.c](https://github.com/rizwijaya/soal-shift-sisop-modul-2-IT01-2021/blob/main/soal3/soal3.c)
### Analisis Soal
Secara keseluruhan, hal yang harus dilakukan dalam soal tersebut adalah:
1. Membuat folder setiap 40 detik dengan nama sesuai timestamp ```[YYYY-mm-dd_HH:ii:ss]```
2. Dari folder tersebut, kemudian mendownload gambar setiap 5 detik dari ```https://picsum.photos/```
3. Setiap gambar berukuran persegi dengan resolusi ```(n%1000) + 50 pixel``` dimana n adalah detik Epoch Unix. Serta diberi nama sesuai dengan timestamp ```[YYYY-mm-dd_HH:ii:ss]```
4. Setelah folder terisi 10 gambar, membuat file ```status.txt``` yang berisi pesan ```Download Success``` yang dienkripsi dengan caesar cipher shift 5
5. Program dijalankan dengan argumen ```-z``` atau ```-x```, dimana argumen pertama akan mengenerate sebuah bash program yang jika dijalankan akan langsung menterminate program, dan argumen kedua jika dijalankan akan menterminate program ketika semua proses sudah selesai (Direktori yang sudah dibuat akan mendownload gambar sampai selesai dan membuat file txt, lalu zip dan delete direktori)
### Penyelesaian
#### Library
Berikut adalah library yang digunakan untuk menyelesaikan soal ini:

```#include <sys/types.h>``` = untuk tipe data ```pid_t```

```#include <sys/stat.h>``` = untuk pengembalian status waktu pada ```time_t```

```#include <stdio.h>``` = untuk standard input-output

```#include <stdlib.h>``` = untuk fungsi-fungsi general

```#include <fcntl.h>``` = untuk file lock dari ```pid_t```

```#include <errno.h>``` = untuk mendefinisikan integer errno, yang diset oleh system call untuk mengindikasikan error

```#include <unistd.h>``` = untuk melakukan system call ```fork()```

```#include <syslog.h>``` = untuk melakukan system log

```#include <string.h>``` = untuk melakukan manipulasi string, misalnya ```strcmp()```

```#include <time.h>``` = untuk mendefinisikan variabel time_t, struct tm, time, localtime, strftime yang bisa manipulasi waktu

```#include <sys/time.h>``` = untuk mendefinisikan timeval structure (tidak dipakai)

```#include <wait.h>``` = untuk melakukan fungsi ```wait ()```

