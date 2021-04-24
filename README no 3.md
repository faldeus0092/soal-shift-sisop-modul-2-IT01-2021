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

#### Fungsi main
Pada fungsi main, program kami menerima 2 argumen. argumen ini nantinya digunakan untuk menentukan bash program mana yang akan dibuat:
```
int main(int argc, char *argv[])
{
  // isi fungsi
}
````

#### Daemon Process
Daemon Process adalah sebuah proses yang bekerja pada background karena proses ini tidak memiliki terminal pengontrol. Process ini kami gunakan karena dapat dijalankan di latar belakang secara terus menerus.

Langkah pertama daemon process adalah melakukan ```fork()``` pada parent process lalu membunuhnya agar sistem mengira proses tersebut selesai.
```
    pid_t process_id = 0;
    pid_t sid = 0;
    // buat child process
    process_id = fork();
    // indikasi fork() gagal
    if (process_id < 0)
    {
        printf("fork failed!\n");
        // Return in exit status
        exit(1);
    }
    // PARENT PROCESS. harus dibunuh.
    if (process_id > 0)
    {
        // return success exit status
        exit(0);
    }
```
Kemudian mengatur umask agar mendapatkan full akses terhadap file
``` umask (0);```

Mengatur SID agar child process tidak menjadi orphan 
```
    if (sid < 0)
    {
        // Return failure
        exit(1);
    }
```

Setelah itu mengubah working directory. Directory berbeda tergantung tiap komputer, jadi sesuaikan sendiri
```
chdir("/home/pepega/sisopShift2/soal3/");
```

Menutup file descriptor standar karena daemon tidak boleh mengakases terminal
```
    close(STDIN_FILENO);
    close(STDOUT_FILENO);
    close(STDERR_FILENO);
```

Terakhir, program utama akan berada di while loop
```
    while (1)
    {
      // program utama
    }
```

#### Pembuatan Bash Program
Pada soal disebutkan untuk membuat bash program sesuai deskripsi sebelumnya. Pertama-tama kami membuat integer untuk menyimpan pid process:
```
    int asdf;
    asdf = getpid();
```

Kemudian untuk memisahkan fungsi antara argumen ```-z``` dan ```-x```, kami menggunakan if else:
```
	if (argc < 2)
	{
		printf("missing argument\n");
		exit(-1);
	}
	if (strcmp(argv[1], "-z") == 0)
	{
		// argumen -z
	}
	else if (strcmp(argv[1], "-x") == 0)
	{
		// argumen -x
	}
```

Untuk membuat bash program, kedua argumen menggunakan logika yang sama:
```
      	FILE *killer = NULL;
	killer = fopen("killer.sh", "w+");
	// permission ganti dengan chmod
	chmod("killer.sh", 0777);
	// nama program sesuaikan dengan nama executable
	fprintf(killer, "isi bash program di sini");
	fflush(killer);
	fclose(killer)
```

Membuat file killer, mengubahnya menjadi mode w+ (untuk read dan write, serta membuat file jika belum ada), mengubah permission dengan chmod. Perbedaannya adalah isi filenya. Untuk argumen ```-z```:

```
  #!/bin/bash
  killall -9 soal3
  rm \"$0\"
```

Untuk argumen ```-x```:

```
  #!/bin/bash
  kill -9 %d
  rm \"$0\"
```

dengan ```%d``` diisi dengan ```int asdf;``` yang berisi PID process.

#### Program Utama
Program ini berada di while loop seperti yang sudah disebutkan sebelumnya. Pertama, kami membuat integer status agar bisa digunakan dalam fungsi wait:
```
	int status;
```

#### Membuat Folder Sesuai Timestamp
Untuk membuat folder sesuai timestamp, kami menggunakan bantuan fungsi ```strftime()``` [referensi bisa dilihat di sini](http://www.cplusplus.com/reference/ctime/strftime/) untuk menyimpan waktu sekarang ke dalam string ```stamp```. Setelah itu menggunakan bantuan ```fork()``` dan ```exec()``` untuk membuat foldernya:

```
        int status;
        int n1 = fork();

        time_t rawtime;
        struct tm *timeinfo;
        char stamp[80];

        time(&rawtime);
        timeinfo = localtime(&rawtime);
        strftime(stamp, 80, "%F_%T", timeinfo);

        if (n1 == 0)
        {
            //Buat folder sesuai waktu sekarang
            char *argv[] = {"mkdir", "-p", stamp, NULL};
            execv("/bin/mkdir", argv);
        }
        else
        {
         // parent process...
        }
```

#### Mendownload Gambar ke dalam Folder
Fungsi untuk mendownload gambar terdapat dalam parent process dari program sebelumnya. Kami menggunakan for loop hingga 10 kali untuk mendownloadnya 10 kali

```
	else
        {
		// while ((wait(&status)) > 0);
		int n8 = fork ();
		if (n8 == 0) {
			for (int i=0; i<10; i++){
				time_t rawtime;
				struct tm *timeinfo;
				char img[80];
				time(&rawtime);
				timeinfo = localtime(&rawtime);
				strftime(img, 80, "%F_%T", timeinfo);
				strcat(img, ".jpg");
				int n3 = fork();

				//ukuran image
				unsigned long size = ((unsigned long)time(NULL) % 1000) + 50;
				//nama image
				char result[80];
				snprintf(result, sizeof(result), "%s/%s", stamp, img);
				//link download
				char url[80];
				snprintf(url, sizeof(url), "https://picsum.photos/%ld/%ld/\?random", size, size);

				if (n3 == 0)
				{
					//Download gambar
					// while ((wait(&status)) > 0);
					execlp("/usr/bin/wget",
							"wget", url, "-q",
							"-O", result, NULL);
				}
				//delay 5 detik
				sleep(5);
			}
			// ...lanjut
        }
```

Untuk mendapatkan nama file, kurang lebih sama seperti nama folder, kami menggunakan fungsi strftime. Kemudian nama image akan disimpan di string ```result```:
```
	time_t rawtime;
	struct tm *timeinfo;
	char img[80];
	time(&rawtime);
	timeinfo = localtime(&rawtime);
	strftime(img, 80, "%F_%T", timeinfo);
	strcat(img, ".jpg");

	// some program ...

	//nama image
	char result[80];
	snprintf(result, sizeof(result), "%s/%s", stamp, img);

	// some program ...
```

Untuk mendapatkan ukuran file, kami menggunakan fungsi ```time(NULL)``` untuk mendapatkan nilai epoch dalam detik. Kemudian dimod dengan 1000 dan ditambah 50 sesuai perintah ~~baginda asisten~~ soal:
```
	//ukuran image
	unsigned long size = ((unsigned long)time(NULL) % 1000) + 50;
```

Untuk mendownload file, kami menggabungkan string url dengan ukuran image menggunakan ```snprintf``` kemudian mendownload gambarnya menggunakan ```wget``` dan ```exec```. Kami berikan ```sleep (5)``` agar program delay selama 5 detik
```
	// some program ...
	//link download
	char url[80];
	snprintf(url, sizeof(url), "https://picsum.photos/%ld/%ld/\?random", size, size);

	if (n3 == 0)
	{
		//Download gambar
		// while ((wait(&status)) > 0);
		execlp("/usr/bin/wget",
				"wget", url, "-q",
				"-O", result, NULL);
	}
	//delay 5 detik
	sleep(5);
```

Kemudian, jika folder sudah terisi 10 gambar, kami membuat sebuah file berisi pesan ```Download Success``` yang dienkripsi dalam caesar cipher shift 5. Untuk pembuatan filenya kami menggunakan logika yang sama seperti file bash tadi. Kami menggunakan fungsi eksternal ```caesarEncryption()``` untuk mengenkripsinya:
```
	//buat txt file
	strcpy(data, "Download Success");
	key = 5;
	caesarEncryption();
	char dir[80];
	snprintf(dir, sizeof(dir), "%s/status.txt", stamp);
	FILE *msg = NULL;
	msg = fopen(dir, "w+");
	fprintf(msg, "%s", data);
	fflush(msg);
	fclose(msg);
```

Adapun fungsi ```caesarEncryption()``` akan menggeser huruf ke kanan sebanyak key (shift) yang diberikan:
```
	//untuk caesar cipher
	char data[50], temp;
	int key, count;

	void caesarEncryption()
	{
	    for (count = 0; data[count] != '\0'; count++)
	    {
		temp = data[count];
		if (temp >= 'a' && temp <= 'z')
		{
		    temp = temp + key;
		    if (temp > 'z')
		    {
			temp = temp - 'z' + 'a' - 1;
		    }
		    data[count] = temp;
		}
		else if (temp >= 'A' && temp <= 'Z')
		{
		    temp = temp + key;
		    if (temp > 'Z')
		    {
			temp = temp - 'Z' + 'A' - 1;
		    }
		    data[count] = temp;
		}
	    }
	}
```

Melakukan zip dan delete dengan bantuan system call ```fork``` dan exec. Ketika menjalankan ini, folder akan dizip lalu dihapus sehingga menyisakan file .zip-nya saja. Jangan lupa menambahkan ```while ((wait(&status)) > 0);``` pada parent process agar penghapusan file berjalan setelah kompres .zip selesai.
```
	//zip file lalu delete
	int n4 = fork();
	if (n4 == 0)
	{
		char zip[80];
		snprintf(zip, sizeof(zip), "%s.zip", stamp);
		execlp("/usr/bin/zip", "zip", "-rm", zip, stamp, NULL);
	}
	else
	{
		while ((wait(&status)) > 0);
		char *deleteAll[] = {"rm", "-rf", stamp, NULL};
		execv("/bin/rm", deleteAll);
	}
```

Terakhir, menambahkan ```sleep (40)``` pada akhir while loop utama agar program dalam while loop berjalan setiap 40 detik.

#### Screenshot
Menjalankan dengan argumen ```-z```
![image](https://user-images.githubusercontent.com/73151823/115948018-ca207680-a4f5-11eb-849a-049bd9abb542.png)

Menjalankan ```killer.sh```
![image](https://user-images.githubusercontent.com/73151823/115948042-e7554500-a4f5-11eb-99fa-7e430feda4ee.png)

Menjalankan dengan argumen ```-x```
![image](https://user-images.githubusercontent.com/73151823/115948158-93972b80-a4f6-11eb-811d-203694426a4d.png)

Menjalankan ```killer.sh```
![image](https://user-images.githubusercontent.com/73151823/115948164-a3af0b00-a4f6-11eb-83fd-72e3b883af37.png)

#### Kendala
Kendala kami adalah ketika membuat program berhenti dengan argumen ```-x```, kami tidak tahu bahwa ternyata harus menggunakan kill dengan PID baru bisa. 
```
int asdf;
asdf = getpid();
snprintf(kille, sizeof(kille), "#!/bin/bash\n\nkill -9 %d\nrm \"$0\"", asdf);
```

Selain itu, untuk memasukkan link ke dalam argumen exec. Ternyata lebih mudah jika menyimpan link tersebut ke dalam string terlebih dahulu
```
	char url[80];
	snprintf(url, sizeof(url), "https://picsum.photos/%ld/%ld/\?random", size, size);

	if (n3 == 0)
	{
		//Download gambar
		// while ((wait(&status)) > 0);
		execlp("/usr/bin/wget",
				"wget", url, "-q",
				"-O", result, NULL);
	}
```

