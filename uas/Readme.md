#Tugas UAS Basis Data Terdistribusi - Wordpress Menggunakan MySQL dan Redis

##Daftar Isi
1. [Model Arsitektur] (#1-model-arsitektur)
2. [Proses Instalasi] (#2-proses-instalasi)
3. [Konfigurasi Wordpress ke Redis] (#3-konfigurasi-wordpress-ke-redis)
4. [Test Failover Redis] (#4-test-failover-redis)

##1. Model Arsitektur
Pada tugas ini, akan digunakan 4 buah server node:

| No | IP Address | Hostname | Deskripsi |
| --- | --- | --- | --- |
| 1 | 192.168.33.11 | clusterdb1 | Sebagai Node Manager dan Redis Master |
| 2 | 192.168.33.12 | clusterdb2 | Sebagai Server 1 dan Node 1 dan Redis Slave 1 |
| 3 | 192.168.33.13 | clusterdb3 | Sebagai Server 2 dan Node 2 dan Redis Slave 2|
| 4 | 192.168.33.14 | clusterdb4 | Sebagai Load Balancer (ProxySQL) |
