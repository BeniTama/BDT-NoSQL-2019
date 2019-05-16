# Tugas Implementasi Redis

## Daftar Isi
1. [Model Arsitektur](#1-model-arsitektur)
2. [Instalasi](#2-instalasi)
3. [Test Redis](#3-test-redis)
4. [Simulasi Fail-over](#4-simulasi-fail-over)

## 1. Model Arsitektur
Pada penugasan implementasi redis ini, akan digunakan 3 buah server node. 1 node akan bertugas sebagai master, dan 2 sebagai slave. Berikut
adalah informasi mengenai node server tersebut:

| No | Alamat IP | Hostname | Peran| RAM |
| --- | --- | --- | --- | --- |
| 1 | 192.168.88.10 | masternode | Master Redis | 2046 MB |
| 2 | 192.168.88.11 | slavenode1 | Slave Redis 1 | 1024 MB |
| 3 | 192.168.88.12 | slavenode2 | Slave Redis 2 | 1024 MB |

File konfigurasi vagrantfilenya adalah sebagai berikut:
```
Vagrant.configure("2") do |config|

	config.vm.define "masternode" do |node|
		node.vm.box = "bento/ubuntu-16.04"
		node.vm.hostname = "masternode"
		node.vm.network "private_network", ip: "192.168.88.10"
		
		node.vm.provider "virtualbox" do |vb|
			vb.gui = false
			vb.name = "masternode"
			vb.memory = "2046"
		end
	end
	
	(1..2).each do |i|
		config.vm.define "slavenode#{i}" do |node|
			node.vm.box = "bento/ubuntu-16.04"
			node.vm.hostname = "slavenode#{i}"
			node.vm.network "private_network", ip: "192.168.88.1#{i}"
			
			node.vm.provider "virtualbox" do |vb|
				vb.gui = false
				vb.name = "slavenode#{i}"
				vb.memory = "1024"
			end
		end
	end
end
```

## 2. Instalasi

Tahapan pertama instalasi adalah melakukan instalasi kebutuhan redis pada semua node:
```
sudo apt-get update
sudo apt-get install build-essential tcl
sudo apt-get install libjemalloc-dev
```

Tahapan berikutnya adalah melakukan instalasi redis dimasing-masing node:
```
curl -O http://download.redis.io/redis-stable.tar.gz
tar xzvf redis-stable.tar.gz
cd redis-stable
make
make test
sudo make install
```

Setelah redis berhasil diinstalasi, dapat dilakukan konfigurasi untuk masing-masing node. Konfigurasi
pertama yang harus dilakukan adalah firewall. Lakukan konfigurasi berikut untuk memperbolehkan koneksi
antar node:
```
sudo ufw allow 6379
sudo ufw allow 26379
sudo ufw allow from 192.168.88.10 (untuk masternode)
sudo ufw allow from 192.168.88.11 (untuk slavenode1) 
sudo ufw allow from 192.168.88.12 (untuk slavenode2)
```

Setelah melakukan konfigurasi pada semua node, lakukan konfigurasi redis dan sentinel pada file 
konfigurasi mereka, yaitu ``redis.conf`` dan ``sentinel.conf``. Untuk melakukannya, dapat diikuti
petunjuk sebagai berikut:
- Masternode redis.conf:
```
protected-mode no
port 6379
dir .
logfile "/home/redis-stable/redig.log" #output log
```
- Slavenode redis.conf:
```
protected-mode no
port 6379
dir .
slaveof 192.168.33.10 6379
logfile "/home/redis-stable/redig.log" #output log
```
Pada ``redis.conf`` sebaiknya dilakukan penghapusan pada line ``bind 127.0.0.1`` agar node mau menerima
koneksi lain selain dari 127.0.0.1. Atau bisa dilakukan seperti ``# bind 127.0.0.1``
- Masternode & Slavenode sentinel.conf:
```
protected-mode no
port 26379
logfile "/home/redis-stable/sentinel.log"
sentinel monitor mymaster 192.168.33.10 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 10000
```

Setelah konfigurasi file konfigurasi berhasil dilakukan, redis dapat dijalankan. Untuk menjalankan
redis dapat dilakukan perintah berikut:
```
src/redis-server redis.conf &
src/redis-server sentinel.conf --sentinel &
```

Kemudian lakukan pengecekan pada status redis masing-masing node:
![](/tugas5-redis/pictures/test-masternode.JPG)
![](/tugas5-redis/pictures/test-slavenode1.JPG)
![](/tugas5-redis/pictures/test-slavenode2.JPG)

Selanjutnya lakukan testing ke masing-masing node menggunakan perintah berikut:
```
redis-cli -h [IP Address] ping
```
Jika ping berhasil, akan keluar output PONG seperti pada gambar berikut:
![](/tugas5-redis/pictures/pong-masternode.JPG)
![](/tugas5-redis/pictures/pong-slavenode1.JPG)
![](/tugas5-redis/pictures/pong-slavenode2.JPG)

Untuk memastikan masing-masing node telah tersinkronisasi dengan baik, dapat diperiksa menggunakan perintah
berikut:
![](/tugas5-redis/pictures/cli-masternode.JPG)
![](/tugas5-redis/pictures/cli-slavenode1.JPG)
![](/tugas5-redis/pictures/cli-slavenode2.JPG)
Disini didapatkan bahwa masternode tersambung kepada dua slave, dan kedua slave menunjukkan master_link_status
aktif. Ini berarti masing-masing node telah tersinkronisasi dengan baik.

## 3. Test Redis

Untuk pengujian redis, master akan melakukan set demokey yang akan di get oleh masing-masing slave.

Perintah untuk melakukan set demokey ada pada gambar dibawah ini pada perintah ``redis-cli`` oleh
masternode:
![](/tugas5-redis/pictures/demokey-masternode.JPG)
Kemudian untuk melakukan get demokey, bisa dilakukan perintah berikut pada salah satu slave.
Jika output yang keluar sesuai, sinkronisasi berhasil dilakukan:
![](/tugas5-redis/pictures/demokey-slavenode.JPG)

## 4. Simulasi Fail-Over

Untuk pengujian fail-over pada node redis, masternode akan dimatikan dengan cara:
```
redis-cli -p 6379 DEBUG sleep 30
```
![](/tugas5-redis/pictures/master-sleep.JPG)

Setelah masternode dimatikan, salah satu node slave akan menjadi masternode. Untuk itu, buka ``redis-cli``
tiap slavenode dan diperiksa node mana yang menjadi master.
Pada kasus ini, slavenode2 yang diangkat menjadi master.
![](/tugas5-redis/pictures/slave-to-master.JPG)

