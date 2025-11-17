# UTS-STDT

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

Dengan kata lain: GraphQL mengorkestrasi IPC secara internal, menggabungkan banyak panggilan antar-proses menjadi satu respons terstruktur untuk client.

---

## Diagram

