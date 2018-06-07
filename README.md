
## Example of using **SPIFFS** with modified esp_idf spiffs VFS driver

**esp-idf** has support for the **spiffs** file system, but the **directory support** is not implemented.

This example uses the modified spiffs VFS driver which enables the directory support.

---

The original esp-idf spiffs driver is modified in a way that directories are enabled.

To enable the new spiffs driver, **spiffs** directory from esp-idf **components** directory is copied to the project's **components** directory.

Only the **esp_spiffs.c** file was modified to enable directory support.

The standard **mkdir()** and **rmdir()** functions are added to the spiffs VFS driver.

The modified version of **mkspiffs** is also provided which supports building spiffs file system image with directories.

---

## Features

* directory handling enabled
* example of directory **list** functions
* example of **file copy** functions

*When using file related functions which has filename argument, prefix* **/spiffs/**  *has to be added to the file name.*

---

## How to build

Configure your esp32 build environment as for other **esp-idf examples**

Clone the repository

`git clone https://github.com/loboris/ESP32_spiffs_example.git`

Execute menuconfig and configure your Serial flash config and other settings. Included *sdkconfig.defaults* sets some defaults to be used.

Navigate to **SPIffs Example Configuration** and set **SPIFFS** options.

Select if you want to use **wifi** (recommended) to get the time from **NTP** server and set your WiFi SSID and password.

`make menuconfig`

Make and flash the example.

`make all && make flash`

---

## Prepare **SPIFFS** image

---

### Build mkspiffs

Before the image can be prepared, the **mkspiffs** executable must be created.<br>
The **sdkconfig.h** is used in build process!<br>
Before building, **copy** the **sdkconfig.h** from **build/include** directory to **components/mkspiffs/include**.<br>
Change the working directory to **components/mkspiffs/include**.<br>
Execute:

```
make clean
make
```

The simple **build_mkspiffs** script is provided which executes all the necessary steps automatically, just execute

```
./build_mkspiffs
```

**Important:** whenever you change spiffs configuration using *menuconfig* you must **rebuild mkspiffs**

---

> **It is not required to prepare the image, as the spiffs will be automatically formated on first use, but it might be convenient.**

<br>

SFPIFFS **image** can be prepared on host and flashed to ESP32.


Copy the files to be included on spiffs into **components/spiffs_image/image/** directory. Subdirectories can also be added.

Execute:

`make makefs`

to create **spiffs image** in build directory **without flashing** to ESP32

Execute:

`make flashfs`

to create **spiffs image** in *build* directory and **flash** it to ESP32

---

<br>

## Example functions

* get the time from NTP server and set the system time (if WiFi is enabled)
* register spiffs as VFS file system; if the fs is not formated (1st start) it will be formated and mounted
* perform some file system tests
  * write text to file
  * read the file back
  * make directory
  * copy file from root to the new directory
  * remove file from new directory
  * remove the new directory
  * list files in root directory and subdirectories
  * perform timing test
  * perform multitask test (if enabled in menuconfig)


---

<br>

**Example output:**

```
I (4221) [SPIFFS example]: ==== STARTING SPIFFS TEST ====

=======================
==== Write to file ====
=======================
  file: "/spiffs/test.txt"
  351 bytes written

========================
==== Read from file ====
========================
  file: "/spiffs/test.txt"
  351 bytes read [
ESP32 spiffs write to file, line 1
ESP32 spiffs write to file, line 2
ESP32 spiffs write to file, line 3
ESP32 spiffs write to file, line 4
ESP32 spiffs write to file, line 5
ESP32 spiffs write to file, line 6
ESP32 spiffs write to file, line 7
ESP32 spiffs write to file, line 8
ESP32 spiffs write to file, line 9
ESP32 spiffs write to file, line 10

]

=================================================
==== Read from file included in sfiffs image ====
=================================================
  file: "/spiffs/spiffs.info"
  405 bytes read [
INTRODUCTION

Spiffs is a file system intended for SPI NOR flash devices on embedded targets.
Spiffs is designed with following characteristics in mind:

  * Small (embedded) targets, sparse RAM without heap
  * Only big areas of data (blocks) can be erased
  * An erase will reset all bits in block to ones
  * Writing pulls one to zeroes
  * Zeroes can only be pulled to ones by erase
  * Wear leveling

]

=============================
==== List root directory ====
=============================

List of Directory [/spiffs/]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
d         -  07/06/2018 10:59  images
f       405  07/06/2018 10:59  spiffs.info
f       351  07/06/2018 10:59  test.txt
-----------------------------------
        756 in 2 file(s)
-----------------------------------
SPIFFS: free 758 KB of 934 KB
-----------------------------------

============================
==== Make new directory ====
============================
  dir: "/spiffs/newdir"
  Directory created
  Copy file from root to new directory...
  List the new directory

List of Directory [/spiffs/newdir]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
f       351  07/06/2018 10:59  test.txt.copy
-----------------------------------
        351 in 1 file(s)
-----------------------------------
SPIFFS: free 757 KB of 934 KB
-----------------------------------

  List root directory, the "newdir" should be listed

List of Directory [/spiffs/]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
d         -  07/06/2018 10:59  images
f       405  07/06/2018 10:59  spiffs.info
f       351  07/06/2018 10:59  test.txt
d         -  07/06/2018 10:59  newdir
-----------------------------------
        756 in 2 file(s)
-----------------------------------
SPIFFS: free 757 KB of 934 KB
-----------------------------------

  Try to remove non empty directory...
  Error removing directory (90) Directory not empty
  Removing file from new directory...
  Removing directory...
  List root directory, the "newdir" should be gone

List of Directory [/spiffs/]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
d         -  07/06/2018 10:59  images
f       405  07/06/2018 10:59  spiffs.info
f       351  07/06/2018 10:59  test.txt
-----------------------------------
        756 in 2 file(s)
-----------------------------------
SPIFFS: free 758 KB of 934 KB
-----------------------------------


================================================
==== List content of the directory "images" ====
==== which is included on spiffs image      ====
================================================

List of Directory [/spiffs/images]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
d         -  07/06/2018 10:59  test
f     39310  07/06/2018 10:59  test1.jpg
f     50538  07/06/2018 10:59  test2.jpg
f     47438  07/06/2018 10:59  test4.jpg
-----------------------------------
     137286 in 3 file(s)
-----------------------------------
SPIFFS: free 758 KB of 934 KB
-----------------------------------

==============================================================
==== Get the timings of spiffs operations
==== Operation:
====   Open file for writting, append 1 byte, close file
====   Open file for readinging, read 8 bytes, close file
==== 2 ms sleep between operations
==== 1000 operations will be executed
==============================================================
I (9028) Test: Started.
I (9498) Test: 100 reads/writes
I (9918) Test: 200 reads/writes
I (10346) Test: 300 reads/writes
I (10779) Test: 400 reads/writes
I (11209) Test: 500 reads/writes
I (11645) Test: 600 reads/writes
I (12079) Test: 700 reads/writes
I (12511) Test: 800 reads/writes
I (12945) Test: 900 reads/writes
I (13376) Test: 1000 reads/writes
W (13376) Test: Min write time: 2523 us
W (13376) Test: Max write time: 2987 us
W (13377) Test: Min read  time: 327 us
W (13382) Test: Max read  time: 462 us
W (13386) Test: Total run time: 4348 ms
I (13391) Test: Finished


List of Directory [/spiffs/]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
d         -  07/06/2018 10:59  images
f       405  07/06/2018 10:59  spiffs.info
f       351  07/06/2018 10:59  test.txt
f      1001  07/06/2018 10:59  testfil3.txt
-----------------------------------
       1757 in 3 file(s)
-----------------------------------
SPIFFS: free 756 KB of 934 KB
-----------------------------------


====================================================
STARTING MULTITASK TEST (3 tasks created)
Operation:
  Open file for writting, append 1 byte, close file
  Open file for readinging, read 8 bytes, close file
2 ms sleep between operations
Each task will perform 1000 operations
Expected run time 40~100 seconds
====================================================
I (14970) [TASK_1]: Started.
I (15170) [TASK_2]: Started.
I (15350) [TASK_1]: 100 reads/writes
I (15370) [TASK_3]: Started.
I (15840) [TASK_2]: 100 reads/writes
I (16220) [TASK_1]: 200 reads/writes
I (16340) [TASK_3]: 100 reads/writes
I (16722) [TASK_2]: 200 reads/writes
I (17149) [TASK_1]: 300 reads/writes
I (17295) [TASK_3]: 200 reads/writes
I (17644) [TASK_2]: 300 reads/writes
I (18055) [TASK_1]: 400 reads/writes
I (18295) [TASK_3]: 300 reads/writes
I (18573) [TASK_2]: 400 reads/writes
I (18999) [TASK_1]: 500 reads/writes
I (19308) [TASK_3]: 400 reads/writes
I (20023) [TASK_2]: 500 reads/writes
I (21294) [TASK_1]: 600 reads/writes
I (21900) [TASK_3]: 500 reads/writes
I (22153) [TASK_2]: 600 reads/writes
I (22577) [TASK_1]: 700 reads/writes
I (22829) [TASK_3]: 600 reads/writes
I (23086) [TASK_2]: 700 reads/writes
I (23541) [TASK_1]: 800 reads/writes
I (23731) [TASK_3]: 700 reads/writes
I (24002) [TASK_2]: 800 reads/writes
I (24417) [TASK_1]: 900 reads/writes
I (24699) [TASK_3]: 800 reads/writes
I (24932) [TASK_2]: 900 reads/writes
I (25308) [TASK_1]: 1000 reads/writes
I (25308) [TASK_1]: Finished
I (25669) [TASK_3]: 900 reads/writes
I (26083) [TASK_2]: 1000 reads/writes
I (26083) [TASK_2]: Finished
I (26875) [TASK_3]: 1000 reads/writes
W (26875) [TASK_3]: Min write time: 2553 us
W (26875) [TASK_3]: Max write time: 154609 us
W (26878) [TASK_3]: Min read  time: 337 us
W (26883) [TASK_3]: Max read  time: 144128 us
W (26888) [TASK_3]: Total run time: 57729 ms
I (26893) [TASK_3]: Finished


List of Directory [/spiffs/]
-----------------------------------
T  Size      Date/Time         Name
-----------------------------------
f      1001  07/06/2018 11:00  testfil2.txt
f       405  07/06/2018 10:59  spiffs.info
f      1001  07/06/2018 11:00  testfil3.txt
d         -  07/06/2018 10:59  images
f      1001  07/06/2018 11:00  testfil1.txt
f       351  07/06/2018 10:59  test.txt
-----------------------------------
       3759 in 5 file(s)
-----------------------------------
SPIFFS: free 754 KB of 934 KB
-----------------------------------


Total multitask test run time: 60 seconds

I (28870) [SPIFFS example]: ==== SPIFFS TEST FINISHED ====

```
