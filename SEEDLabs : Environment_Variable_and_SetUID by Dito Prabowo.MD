## SEEDlabs: Environment Variable and Set-UID Lab By Dito Prabowo

### 0x00 Intro

Tujuan dari pembelajaran lab ini adalah agar kita bisa mengerti bagaimana Environment variable bisa mempengaruhi program dan system. Environment variable adalah set dari nilai dinamic yang bisa mempengaruhi sebuah proses yang berjalan di computer. Pada lab ini kita akan mengetahui bagaimana Environment variable berjalan, bagaimana bisa diperbanyak atau diadaptasi dari parent proses (program utama/inti) ke child proses (proses turunan dari inti) dan bagaimana pengaruhnya dalam system / program.

### 0x01 Lab Tasks

#### Task 1 : Manipulating Environment variable
Dalam Task ini, kita belajar untuk menampilkan Environment, perintah export dan unset.

Menampilkan Environment : gunakan perintah    

    printenv

atau

    env

Hasil :
![printenv atau env command](images/task1.png)

untuk Menampilkan env tertentu misalnya SESSION kita bisa gunakan perintah

    printenv SESSION

atau

    env | grep SESSION

Hasil : ![printenv session](images/printenvsession.png)

Untuk menambahkan kita bisa menggunakan perintah :

    export

dan untuk menghilangkan :

    unset

Hasil :

![export](images/exportlog.png)

![Show Log](images/showlog.png)

![unset](images/loghilang.png)

***

### TASK 2 : Inheriting environment variables from parents

Dalam Task ini, kita belajar bagaimana environment variable diturunkan di child proses dari parent proses.
Dalam Unix, fork() membuat proses baru dengan menduplikat proses utama. Proses baru( child proses ) sepenuhnya persis dengan proses utama, tapi ada beberapa yang tidak diturunkan ke child ( untuk mengetahuinya kita bisa lihat manual page fork di _man fork_ ), dalam task ini kita ingin mengetahui apakah parent environment dapat diturunkan di child atau tidak.

#### Step 1

Karena output mengandung banyak strings, kita simpan saja output ke dalam file dengan tanda ">" yang artinya menyimpan hasil ke dalam file tertentu sesuai nama yang kita berikan.

```objective-c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>

extern char **environ;

void printenv()
{
    int i = 0;
    while(environ[i] != NULL) {
        printf("%s\n", environ[i]);
        i++;
    }
}

void main()
{
    pid_t childPid;

    switch(childPid = fork()) {
        case 0:    // child process
            printenv();
            exit(0);
        default:   // parent process
            // printenv();
            exit(0);
    }
}
```

Hasil :

![step1](images/step1.png)

#### Step 2
Sekarang comment printenv() pada bagian child, dan uncomment printenv pada bagian parent, compile dan jalankan. simpan hasil kedalam file yang berbeda.


![step2](images/step2.png)
note : pada step2.c sudah dilakukan perintah step2

My observation : pada step 1 kita menampilkan environtment pada proses child dan pada step 2 kita menampilkan environment pada parent proses
#### Step 3

pada step ini, kita menampilkan perbedaan kedua hasil dengan command diff.

Hasil :
![step3](images/diff.png)

Penjelasan : pada diff ada spesial simbol diantaranya

    a : add
    c : change
    d : delete
    * Lines preceded by a < are lines from the first file.
    * Lines preceded by > are lines from the second file.
    * The three dashes (“—“) merely separate the lines of file 1 and file 2.

Pada hasil diatas berarti kita mendapatkan hasil yang sama kecuali pada line 75 ada perbedaan yaitu pada file pertama "_=./step1" dan pada file kedua "_=./step2", yaitu argumen untuk menjalankan program
***

### Task 3 :  Environment Variables and execve()

Dalam task ini, kita belajar  bagaimana environment variabel mempengaruhi ketika program baru di eksekusi dengan execve(). fungsi execve() memanggil system call untuk memanggil command baru dan mengeksekusinya. fungsi ini tidak pernah kembali. Tidak ada proses baru yang dibuat. dan the calling process's text, data, bss, and stack di overwrite oleh program yang dipanggil. Intinya, execve() menjalankan program baru dalam pemanggilan proses. dalam environment variabel apakah otomatis diturunkan oleh program baru?

#### Step 1

Compile dan jalankan program dibawah ini. program ini mengeksekusi program yang memanggil /usr/bin/env, yang menampilkan environment variabel dari proses yang dijalankan.

```objective-c
#include<stdio.h>
#include<stdlib.h>

extern char **environ;

int main()
{
    char *argv[2];

    argv[0] = "/usr/bin/env";
    argv[1] = NULL;

    execve("/usr/bin/env", argv, NULL);

    return 0;
}
```
Hasil :
![task3](images/task3.png)

** Hasil : Tidak ada output

### Step 2
Ganti code execve diatas dengan code seperti dibawah

    execve("/usr/bin/env", argv, environ);

![step2task3](images/step2task3.png)

hasilnya keluar environment variabel

Penjelasan :
setelah membaca man execve, ada 3 parameter pada execve yaitu :

    int execve(const char *filename, char *const argv[],
    char *const envp[]);

** pada saat envp[] null,tidak ada yang ditampilkan.
tapi  pada saat envp[] diisi environ, memunculkan semua envriontment.

    extern char **environ;

berarti code diatas meload environtment variabel dan mengeksekusinya dengan execve , dan env tidak bisa langsung dipanggil tanpa meload environment variabel.

-**
***

### Task 4: Environment Variables and system()

Dalam Task ini, kita mengeksekusi environment variabel dengan system(),dengan func system() environtment variabel bisa terload  karena system() menggunakan execl() untuk mengeksekusi /bin/sh; kemudian execl() memanggil execve() yang telah melewatkan envriontment variabel array.
jadi tidak perlu me load environment variabel secara manual

```objective-c
#include<stdio.h>
#include<stdlib.h>

int main()
{
    system("/usr/bin/env");

    return 0;
}
```

hasil :
![task4](images/task4.png)

***
### Task 5: Environment variable and Set-UID Programs

Set-UID sangat penting dalam mekanisame keamanan di unix.
ketika set-UID program berjalan, program itu akan menjalankan program sebagai pemilik aslinya.
Contohnya, ada program yang privilage nya root, maka siapapun yang menjalankan program set-UID akan mendapat akses root(dalam program ini) dalam mengeksekusinya.
dalam lab ini kita akan mencoba memahami pengaruhnya set-UID dalam environment variabel.

#### Step 1

 kita buat program untuk menampilkan environment variabel

 ```objective-c
 #include<stdio.h>
 #include<stdlib.h>

 extern char **environ;

 void main()
 {
     int i = 0;
     while(environ[i] != NULL) {
         printf("%s\n", environ[i]);
         i++;
     }
 }
 ```

#### Step2
kita compile program diatas dan mengganti ownership ke root, dan membuat set-UID program.

![task5](images/task5.png)

#### Step3
Dalam Bash shell (normal account), gunakan export untuk mengeset beberapa environment variabel

* PATH
* LD_LIBRARY_PATH
* ANY_NAME

![task5_3](images/task5_2.png)

Dalam PATH dan LD..
ditambahkan :$[name] karena agar value sebelumnya tidak hilang, jika hilang bisa menyebabkan bbrapa command tidak bisa dieksekusi langsung, harus memanggil sampek direktori asalnya, misalnya date harus memanggil dengan /bin/date

Lalu eksekusi :

![task5ke3](images/task5_3.png)
