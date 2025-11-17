# UTS-STDT
# NIM : 245410076
# NAMA : Bayu Apriansyah Putra

## 1. Jelaskan teorema CAP dan BASE dan keterkaitan keduanya. Jelaskan menggunakan contoh yang pernah anda gunakan.

**CAP theorem (Brewer)**  
Dalam sistem terdistribusi, tiga properti berikut tidak bisa semuanya dijaga sempurna pada saat terjadi partition (hubungan jaringan terputus):

- **C = Consistency:** Setelah operasi selesai, semua node melihat data yang sama (sangat kuat).  
- **A = Availability:** Setiap permintaan akan mendapatkan respons (tidak error).  
- **P = Partition tolerance:** Sistem tetap berfungsi walau ada partisi jaringan.

Intinya: saat ada partition, harus memilih antara **Consistency** atau **Availability** (P sudah harus ditolerir karena jaringan tidak sempurna).

---

**BASE**  
Filosofi yang sering dipakai pada sistem eventually consistent (terutama NoSQL, caching, dsb):

- **B = Basically Available** (tersedia secara dasar)  
- **A = Soft-state** (state bisa berubah seiring waktu tanpa input baru)  
- **SE = Eventual consistency** (akhirnya konsisten)

---

**Keterkaitan CAP dan BASE:**  
BASE adalah pendekatan praktis ketika memilih **AP (Availability + Partition tolerance)** dalam CAP: sistem memilih tetap melayani permintaan meski konsistensi langsung tidak dijamin, dengan janji *eventual consistency*.  

Sebaliknya, kalau memilih **CP**, sistem akan lebih ketat pada konsistensi (misalnya menggunakan locking, quorum writes) dan mungkin menolak atau menunda operasi saat partisi (mengorbankan availability).

---

**Contoh konteks (menggunakan project Sistem Streaming Replication + PostgreSQL yang pernah dibuat):**  
Sistem Streaming Replication PostgreSQL 18 merupakan contoh nyata penerapan CAP & BASE. Pada konfigurasi default (asynchronous), cluster PostgreSQL memilih **AP** dan mengikuti prinsip BASE: sistem tetap available meskipun data antar node belum konsisten, dan akan konsisten pada akhirnya (*eventual consistency*).  

Apabila diubah menjadi synchronous replication, PostgreSQL berubah menjadi sistem **CP**, yaitu mengutamakan konsistensi dengan mengorbankan availability. Dengan demikian, proyek streaming replication tersebut sangat tepat sebagai ilustrasi bagaimana CAP dan BASE diterapkan pada sistem terdistribusi modern.

---

## 2. Jelaskan keterkaitan antara GraphQL dengan komunikasi antar proses pada sistem terdistribusi. Buat diagramnya.

Keterkaitan GraphQL dengan komunikasi antar proses pada sistem terdistribusi terletak pada perannya sebagai **API Gateway** yang bertugas menghubungkan client dengan berbagai microservice di backend.  

Ketika client mengirim satu query GraphQL, server GraphQL tidak hanya memprosesnya secara lokal, tetapi menjalankan beberapa komunikasi antar proses (IPC) ke berbagai service berbeda seperti User Service, Order Service, atau Inventory Service.

Setiap resolver dalam GraphQL dapat berhubungan dengan proses lain melalui:

- HTTP REST  
- gRPC  
- Message queue  
- Database call  

ini arinya GraphQL mengorkestrasi IPC secara internal, menggabungkan banyak panggilan antar-proses menjadi satu respons terstruktur untuk client.

---

## Diagram
               Client
                  │
                  │ Query GraphQL
                  ▼
         GraphQL Server (API Gateway)
             ┌───────────────┬───────────────┐
             │               │               │
             ▼               ▼               ▼
      User Service     Order Service    Inventory Service
    (IPC: REST/gRPC) (IPC: REST/gRPC) (IPC: REST/gRPC)


## 3. Dengan menggunakan Docker / Docker Compose, buatlah streaming replication di PostgreSQL yang bisa menjelaskan sinkronisasi. Tulislah langkah-langkah pengerjaannya dan buat penjelasan secukupnya.

# Streaming Replication PostgreSQL 18 dengan Docker & Docker Compose

Repository ini berisi konfigurasi lengkap untuk membangun **Streaming Replication** pada PostgreSQL 18 (alpine) menggunakan Docker Compose.  
Terdapat **Primary Node** dan **Replica Node** yang tersinkronisasi secara real-time menggunakan **physical replication** + **replication slot**.

---

## Struktur Folder
streaming-replication-uts/
│── 00_init.sql
│── docker-compose.yaml
└── env.sh



---

# Penjelasan Streaming Replication

**Streaming replication** adalah metode replikasi PostgreSQL yang mengirim WAL (Write-Ahead Log) secara kontinu dari primary ke replica.

Kelebihannya:

- Sinkronisasi real-time  
- Read-only query dari replica  
- High Availability (HA)  
- WAL aman berkat replication slot  

---

# 2️ Penjelasan File

## 00_init.sql

Script inisialisasi yang dijalankan ketika PostgreSQL primary pertama kali dibuat.

```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');

```
Fungsi:
Membuat user khusus untuk replikasi
 dan Membuat replication slot agar WAL tidak hilang sebelum dikonsumsi replica

 ## docker-compose.yaml

Terdiri dari dua service utama:
postgres_primary
Konfigurasi utama:

Port: 5432

Init script otomatis dijalankan

Parameter replikasi:
```
wal_level=replica
max_wal_senders=10
max_replication_slots=10
hot_standby=on
hot_standby_feedback=on
```

postgres_replica

Fungsi: mengambil backup dari primary lalu masuk mode streaming.

Menggunakan:
```
pg_basebackup -R --slot=replication_slot --host=postgres_primary --port=5432 -X stream
```
Hasil: node menjadi hot standby.

## env.sh

Alias command untuk mempermudah:
```
alias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias der="sudo docker exec -it streaming-replication-uts_postgres_replica_1 bash"
alias dep="sudo docker exec -it streaming-replication-uts_postgres_primary_1 bash"
alias dlr="sudo docker logs streaming-replication-uts_postgres_replica_1"
alias dlp="sudo docker logs streaming-replication-uts_postgres_primary_1"
```

lalu aktifkan:
```
source env.sh
```
# Menjalankan Replikasi

Menjalankan container
```
dcu
```
Mengecek container
```
dps
```

Akan muncul:

maka akan muncul postgres_primary dan postgres_replica

Mengecek Status Replikasi
Mengecek dari PRIMARY

Masuk:

dep
psql -U zuser -d zdb


Cek:

SELECT * FROM pg_stat_replication;


Jika berhasil → state = streaming.

# Mengecek dari REPLICA

Masuk:
```
der
psql -U zuser -d zdb
```

Cek apakah sedang recovery:
```
SELECT pg_is_in_recovery();
```

Hasil yang benar:

```
t
```

# Pengujian Sinkronisasi
 Buat tabel dan insert di PRIMARY
 ```
CREATE TABLE test_replica (first INT, second VARCHAR(15));
INSERT INTO test_replica VALUES (10, 'TEST REPLICA');
INSERT INTO test_replica VALUES (10, 'TEST REPLICA 2');
```
Cek di REPLICA
```
SELECT * FROM test_replica;
```

Jika datanya sama → Replikasi berhasil.

 # Failover: Promote Replica
* Stop primary
```
sudo docker-compose stop postgres_primary
```

Promote replica menjadi primary baru
```
SELECT pg_promote();
```

Cek:
```
SELECT pg_is_in_recovery();
```

Jika hasil = f → Replica telah menjadi Primary.

* Kesimpulan

Primary dan Replica tersinkronisasi melalui WAL streaming

Replica dapat mengambil alih menjadi Primary (failover)

Docker Compose mempermudah proses setup cluster PostgreSQL untuk belajar/pengujian

Replication slot memastikan WAL tidak hilang sebelum dikonsumsi



