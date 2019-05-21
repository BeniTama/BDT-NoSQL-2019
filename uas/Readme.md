# Tugas UAS Basis Data Terdistribusi - Wordpress Menggunakan MySQL dan Redis

## Daftar Isi
1. [Model Arsitektur](#1-model-arsitektur)
2. [Proses Instalasi](#2-proses-instalasi)
3. [Konfigurasi Redis pada Wordpress](#3-konfigurasi-redis-pada-wordpress)

## 1. Model Arsitektur
Pada tugas ini, akan digunakan 4 buah server node:

| No | Alamat IP | Nama Node | Deskripsi |
| --- | --- | --- | --- |
| 1 | 192.168.33.11 | clusterdb1 | Node Manager dan Redis Master |
| 2 | 192.168.33.12 | clusterdb2 | Server 1 dan Slave redis 1 |
| 3 | 192.168.33.13 | clusterdb3 | Server 2 dan slave redis 2|
| 4 | 192.168.33.14 | clusterdb4 | Load Balancer |

## 2. Proses Instalasi

Awal dari instalasi tugas ini, akan dibuat 4 buah node untuk MySQL Cluster. Untuk melihat tata cara instalasi dari MySQL Cluster, dapat dilihat pada dokumentasi tugas berikut: 

[implementasi-mysql_cluster]()

Setelah melakukan instalasi MySQL Cluster, instalasi Wordpress dapat dilakukan. Dokumentasi tahapan instalasi terdapat pada link berikut:

[mysql-cluster-wordpress]()

Setelah instalasi Wordpress berhasil dilakukan, selanjutnya dilakukan instalasi Redis pada node clusterdb1 sebagai master dan clusterdb2 & clusterdb3 sebagai slave redis. Tata cara instalasi redis dapat ditemukan dalam dokumentasi berikut:

[Tugas 5: Redis](https://github.com/BeniTama/BDT-NoSQL-2019/tree/master/tugas5-redis)

Setelah Redis berhasil dikonfigurasi, selanjutnya adalah melakukan konfigurasi redis pada Wordpress.

## 3. Konfigurasi Redis pada Wordpress

Untuk melakukan konfigurasi Redis pada Wordpress, langkah pertama adalah melakukan instalasi pada plugin ``Wordpress Redis Object Cache``. Cara melakukan instalasi plugin ini adalah dengan memasuki tab ``Add New`` dalam ``Plugin`` kemudian melakukan pencarian dengan keyword redis. Setelah itu, lakukan instalasi plugin yang ada dalam gambar berikut:

![](/uas/pictures/redis-plugin-install.png)

Setelah instalasi plugin berhasil dilakukan, selanjutnya adalah melakukan konfigurasi pada ``wp_config.php`` yang terletak pada clusternode4. Konfigurasinya seperti yang ada dalam gambar berikut. Lokasi penulisan konfigurasi juga harus tepat, yaitu dibawah konfigurasi DB.

![](/uas/pictures/wp-config-configuration.JPG)

Setelah konfigurasi berhasil dilakukan, kembali ke tab ``Plugin`` atau ``Installed Plugin`` dan lakukan aktivasi pada plugin dengan menekan tombol ``Activate`` dibawah nama plugin.

![](/uas/pictures/install_roc.png)

Untuk memastikan bahwa redis terhubung dengan Wordpress, pilih pilihan ``Settings`` pada plugin dan pastikan bahwa status plugin adalah ``Connected``.

![](/uas/pictures/redis-wordpress.JPG)




