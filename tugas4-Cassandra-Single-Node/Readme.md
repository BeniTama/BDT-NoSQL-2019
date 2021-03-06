# Cassandra Single Node
Implementasi instalasi Cassandra Single Node & Operasi CRUD yang dilakukan

## Daftar Isi
1. [Pengantar](#1-pengantar)
    -   [Apa Itu Cassandra](#11-apa-itu-cassandra)
    -   [Arsitektur Server](#12-arsitektur-server)
2. [Instalasi](#2-instalasi)
    -   [Konfigurasi Vagrantfile](#21-konfigurasi-vagrantfile)
    -   [Instalasi Oracle Java Virtual Machine](#22-instalasi-oracle-java-virtual-machine)
    -   [Instalasi Cassandra](#23-instalasi-cassandra)
3. [Pemilihan Dataset](#3-dataset)
4. [CRUD Data](#4-crud)
	- [Import Dataset](#41-impor-dataset)
	- [Melihat Data](#42-melihat-data)
	- [Membuat Entry Data](#43-membuat-data)
	- [Memodifikasi/Memperbaharui Data](#44-mengubah-data)
	- [Menghapus Data](#45-menghapus-data)
5. [Referensi](#5-referensi)

## 1. Pengantar
### 1.1 Apa Itu Cassandra
Cassandra adalah sebuah sistem manajemen basis data yang didesain untuk mengelola data dengan jumlah yang sangat besar yang tersebar dalam beberapa server, membuat layanan tetap tersedia tanpa sebuah titik kegagalan/single point of failure (SPOF). Cassandra menawarkan dukungan yang kuat pada cluster yang menjangkau beberapa datacenter.

Cassandra adalah salah satu implementasi dari NoSQL (Not Only SQL). NoSQL merupakan konsep penyimpanan database dinamis yang tidak terikat pada relasi-relasi tabel yang kaku seperti RDBMS. Selain lebih scalable, NoSQL juga memiliki performa pengaksesan yang lebih cepat. NoSQL tidak membutuhkan data model relational dan bahasa SQL untuk menyimpan atau mengambil data dari database. NoSQL menggunakan metadata pada database dan memanfaatkan index dari data tersebut.

Perbandingan antara SQL dengan NoSQL dapat dibaca dari tabel dibawah ini:
![Tabel Perbedaan MySQL dan NoSQL](/tugas4-Cassandra-Single-Node/pictures/perbedaan-mysql-nosql.PNG)
<br>
Sementara perbandingan antara Apache Cassandra dengan database NoSQL lainnya, semisal MongoDB, adalah seperti dalam tabel berikut:
![Tabel Perbedaan Cassandra dengan MongoDB](/tugas4-Cassandra-Single-Node/pictures/perbedaan-cassandra-mongodb.PNG)

### 1.2 Arsitektur Server
Arsitektur server untuk tugas implementasi ini sangatlah simpel, yaitu hanya menggunakan satu buah node untuk Cassandra dan data yang akan diolah.

![Arsitektur Node](/tugas4-Cassandra-Single-Node/pictures/arsitektur.PNG)

## 2. Instalasi
### 2.1 Konfigurasi Vagrantfile
Konfigurasi vagrantfile untuk tugas ini sebagai berikut:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
	config.vm.define "node1" do |node|
		# Box Settings
		node.vm.box = "bento/ubuntu-16.04"
		node.vm.hostname = "node1"

		# Provider Settings
		node.vm.provider "virtualbox" do |vb|
		# Display the VirtualBox GUI when booting the machine
		vb.gui = false
		vb.name = "node1"
		# Customize the amount of memory on the VM:
		vb.memory = "512"
		end

		# Network Settings
		# slave.vm.network "forwarded_port", guest: 80, host: 8080
		# slave.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
		node.vm.network "private_network", ip: "192.168.33.11"
		# slave.vm.network "public_network"

		# Folder Settings
		# slave.vm.synced_folder "../data", "/vagrant_data"

		# Provision Settings
		# slave.vm.provision "shell", inline: <<-SHELL
		#   apt-get update
		#   apt-get install -y apache2
		# SHELL
	end
end
```
Dari vagrantfile ini dapat dijelaskan bahwa hanya akan dibuat satu node saja, yang menggunakan OS Ubuntu 16.

### 2.2 Instalasi Oracle Java Virtual Machine
Untuk dapat melakukan instalasi, Cassandra membutuhkan Oracle Java SE Runtime Environment (JRE). Maka dari itu, perlu dilakukan instalasi JRE. Java yang dibutuhkan dalam Cassandra adalah Java versi 8. Cassandra belum dapat mensupport Java versi terbaru.

Untuk melakukan instalasi Oracle JRE, pertama-tama harus ditambahakan Personal Package Archive menggunakan perintah berikut:
```
sudo add-apt-repository ppa:webupd8team/java
```
Akan ada konfirmasi untuk melanjutkan instalasi atau tidak. Tekan ``Enter`` untuk melanjutkan.
Kemudian lakukan update:
```
sudo apt-get update
```
Setelah itu lakukan instalasi Oracle JRE.
```
sudo apt-get install oracle-java8-set-default
```
Lakukan perintah berikut untuk mengecek versi Java yang sudah terinstall:
```
java -version
```
Output yang didapatkan seharusnya mirip seperti dibawah ini:
![Versi Java](/tugas4-Cassandra-Single-Node/pictures/java-version.PNG)

### 2.3 Instalasi Cassandra
Untuk instalasi Cassandra akan digunakan paket terakhir dari repositori Apache Software Foundation. Jadi langkah pertama yang dilakukan adalah menambahkan repo mereka agar paketnya dapat digunakan. Cassandra yang dilakukan pada pembuatan proyek ini adalah versi 39x. Gunakan perintah ini untuk menambahkan repo:
```
echo "deb http://www.apache.org/dist/cassandra/debian 39x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
echo "deb-src http://www.apache.org/dist/cassandra/debian 39x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
```
Untuk menghindari warning yang bisa didapatkan ketika melakukan update, beberapa public key dari Apache Software Foundation perlu ditambahkan. Untuk versi ``39x``, hanya ada 2 public key yang perlu ditambahkan. Tambahkan kunci dengan perintah sebagai berikut:
```
gpg --keyserver pgp.mit.edu --recv-keys 0353B12C
gpg --export --armor 0353B12C | sudo apt-key add -
```
Jika penambahan key berhasil maka akan muncul hasil sebagai berikut:
![public key berhasil](/tugas4-Cassandra-Single-Node/pictures/add-key.PNG)

Untuk Cassandra versi 39x, perlu ditambahkan satu kunci tambahan. Jika kunci itu belum ditambahkan maka akan terjadi warning ketika melakukan update. Tambahkan kunci itu dengan perintah sebagai berikut:
```
sudo apt-key adv --keyserver pool.sks-keyservers.net --recv-key A278B781FE4B2BDA
```
Perintah berikut akan memunculkan keluaran seperti gambar berikut:
![Troubleshoot public key](/tugas4-Cassandra-Single-Node/pictures/key-troubleshoot.PNG)
Dapat dilihat pada gambar jika kunci belum ditambahkan sebuah warning akan muncul saat melakukan update. Setelah kunci ditambahkan, lakukan update lagi dan warning tidak akan muncul kembali.
![Troubleshoot done](/tugas4-Cassandra-Single-Node/pictures/troubleshoot-done.PNG)

Setelah update berhasil dilakukan, instalasi Cassandra bisa dilakukan.
```
sudo apt-get install cassandra
```

Status Cassandra dapat dilihat dengan perintah berikut:
```
sudo service cassandra status
```
Hasil yang muncul jika instalasi sukses dan Cassandra berjalan adalah seperti berikut:
![](/tugas4-Cassandra-Single-Node/pictures/Capture.PNG)

Setelah instalasi sukses dan Cassandra berjalan, status node pun bisa dicek dengan perintah sebagai berikut:
```
sudo nodetool status
```
Hasilnya seperti gambar berikut:
![](/tugas4-Cassandra-Single-Node/pictures/nodetool-berhasil.PNG)
Ini adalah bukti bahwa Cassandra berhasil berjalan pada node node1.

## 3. Pemilihan Dataset
Disini akan dilakukan pemilihan dataset untuk melakukan operasi CRUD pada node1. Dataset yang akan digunakan berasal dari www.kaggle.com, dan dataset yang digunakan adalah "Japan Hostel Dataset". 
![](/tugas4-Cassandra-Single-Node/pictures/dataset.PNG)
Dataset ini berisi informasi mengenai lebih dari 300 hostel di Japan. Didalam dataset ini ada informasi mengenai nama hostel, kota, harga minimum, jarak dari tengah kota, nilai rating, fasilitas, dll.
Dataset ini dipilih karena ukurannya yang kecil dan jumlah kolomya mempermudah operasi CRUD yang akan dilakukan nanti.

## 4. CRUD Data
### 4.1 Impor Dataset
Untuk melakukan impor data, salin file csv yang ingin diimpor dari folder vagrant ke node yang digunakan. Untuk menyalin file, gunakan perintah berikut:
```
sudo cp '/vagrant/Hostel.csv' 'Hostel.csv'
```
Perintah ini akan memindahkan file yang bernama 'Hostel.csv' dari folder vagrant yang digunakan ke direktori awal dari node server. File yang akan tersalin akan dinamakan 'Hostel.csv'.

Setelah file csv berhasil tersalin dalam node1, operasi impor data dan CRUD dapat dijalankan dalam CQL. Untuk mengakses CQL gunakan command ``cqlsh``.
Setelah berhasil masuk ke CQL, harus dibuat keyspace untuk lokasi tabel dimana dataset akan diimpor. Untuk membuat keyspace gunakan perintah seperti berikut:
```
CREATE KEYSPACE hostel
  WITH REPLICATION = {
  'class' : 'NetworkTopologyStrategy',
  'datacenter1' : 1
};
```
Setelah keyspace dibuat gunakan keyspace dengan perintah ``USE hostel``.
Hasil dari operasi login ke cqlsh dan pembuatan keyspace seperti gambar dibawah ini:
![](/tugas4-Cassandra-Single-Node/pictures/create-keyspace.PNG)

Setelah menggunakan keyspace yang sudah dibuat. Impor file csv dapat dilakukan. Lakukan impor dengan perintah berikut:
```
COPY hostel_list(id, hostel_name, city, price_from, distance, summary_score, rating_band, atmosphere, cleanliness, facilities, location_y, security, staff, valueformoney, lon, lat) FROM 'Hostel.csv' WITH DELIMITER = ',' AND HEADER = TRUE;
```
Jika muncul keluaran seperti gambar dibawah ini artinya impor data telah berhasil.
![](/tugas4-Cassandra-Single-Node/pictures/import-table.PNG)

### 4.2. Melihat Data
Untuk melihat semua data yang sudah diimpor lakukan perintah berikut:
```
SELECT * FROM hostel_list;
```
Keluaran yang ada seperti gambar dibawah ini:
![](/tugas4-Cassandra-Single-Node/pictures/import-complete.PNG)

### 4.3 Membuat Data
Berikut adalah percobaan memasukkan data baru pada tabel. Data yang akan dimasukkan adalah seperti berikut:
```
id = 343
hostel_name = Hostel Nakamura
City = Kobe
price.from = 2300
Distance = 2.3km from city centre
summary_score = 8.5
rating.band = Fabulous
atmosphere = 7.8
cleanliness = 8.1
facilities = 7.8
location.y = 9.0
security = 8.7
staff = 9.1
valueformoney = 9.0
lon = 135.1731553
lat = 34.677261
```
Data berikut akan dimasukkan kedalam tabel dengan perintah berikut:
```
INSERT INTO hostel_list(id, hostel_name, city, price_from, distance, summary_score, rating_band, atmosphere, cleanliness, facilities, location_y, security, staff, valueformoney, lon, lat) VALUES (343, 'Hostel Nakamura', 'Kobe', '2300', '2.3km from city centre', '8.5', 'Fabulous', '7.8', '8.1', '7.8', '9.0', '8.7', '9.1', '9.0', '135.1731553', '34.677261');
```
Setelah itu select data yang baru dimasukkan untuk melihat datanya berhasil masuk atau tidak, seperti pada gambar berikut:
![insert-data](/tugas4-Cassandra-Single-Node/pictures/insert-data.PNG)

### 4.4 Mengubah Data
Pada bagian ini akan dijelaskan proses modifikasi data dalam Cassandra. Data yang akan dimodifikasi adalah pada entri dengan id 91, yaitu pada entri Guest House Hokorobi. Entri data dalam sampel ini belum memiliki garis lintang dan garis bujur. Percobaan kali adalah untuk menambahkan data lintang dan bujur tempat itu.

Data yang akan ditambahkan adalah sebagai berikut:
```
lat = 33.57771
lon = 130.3364923
```

Update data dilakukan dengan perintah berikut:
```
UPDATE hostel_list SET lat = '33.57771', lon = '130.3364923' WHERE id=91;
```
Hasilnya akan seperti berikut:
![update-complete](/tugas4-Cassandra-Single-Node/pictures/update-complete.PNG)

### 4.5 Menghapus Data
Pada bagian ini akan dijelaskan proses penghapusan data dalam Cassandra. Data yang akan dihapus adalah data dengan entri id 214. Untuk melakukan penghapusan data lakukan perintah berikut:
```
DELETE FROM hostel_list WHERE id = 214;
```

## 5. Referensi
https://www.digitalocean.com/community/tutorials/how-to-install-cassandra-and-run-a-single-node-cluster-on-ubuntu-14-04<br>
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04<br>
http://cassandra.apache.org/download<br>
https://medium.com/@danairwanda/pengenalan-cassandra-database-nosql-3d33a768a20<br>
https://scalegrid.io/blog/cassandra-vs-mongodb/<br>

