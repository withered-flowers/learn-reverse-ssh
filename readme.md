# Mengenal Reverse SSH dan Cara Penggunaannya

## Table of Content
1. [Persyaratan & Asumsi](#persyaratan-&-asumsi)
1. [Apa itu SSH](#apa-itu=ssh)
1. [Apa itu Reverse SSH Tunneling](#apa-itu-reverse-ssh-tunneling)
1. [Apa itu ngrok](#apa-itu-ngrok)
1. [PoC & Showcase](#poc-&-showcase)
1. [Referensi](#referensi)

## Persyaratan & Asumsi
Dalam pembelajaran ini, Anda diminta sudah:
- Terbiasa melihat layar hitam putih (terminal / command-line)
- Mengenal perintah dasar dalam terminal yang digunakan
- Memiliki pengetahuan dasar mengenai jaringan 
  (karena terminologi mengenai jaringan tidak dibahas lebih lanjut di pembelajaran ini)
- Mengerti menggunakan ssh dan setup (instalasi) sshd di komputer sendiri
  (tidak diajarkan instalasi ssh di sini)

## Apa itu SSH
SSH atau *secure shell* merupakan sebuah protokol jaringan kriptografi untuk komunikasi data yang
aman yang berfungsi sebagai pengeksekusi perintah jarak jauh *remote* antar dua jaringan
komputer.

SSH ini sendiri dapat digunakan untuk berbagai aplikasi, sebagai contoh:
- dalam kombinasi dengan SFTP, sebagai alternatif yang cukup aman untuk melakukan transfer file 
  via FTP
- untuk browsing web melalui koneksi proxy yang dienkripsi klien SSH yang mendukung SOCKS
- untuk otomasi remote monitoring dan pengelolaan server 
- untuk melakukan *port forwarding* atau *tunneling port*
- dan masih banyak lainnya lagi ...

## Apa itu Reverse SSH Tunneling
Diketahui dari poin sebelumnya, bahwa salah satu kegunaan dari SSH adalah untuk melakukan
*port forwarding* atau untuk melakukan *tunneling port*

Dalam suatu kasus, kita diminta untuk melakukan melakukan SSH untuk mengontrol suatu komputer
jarak jauh yang berfungsi sebagai server, hanya saja ternyata server tersebut ada di dalam suatu
jaringan yang cukup *merepotkan*, mis: dalam NAT atau dalam aturan firewall yang cukup kompleks. 

Bagaimanakah cara kita mengkoneksikan komputer yang ada di tempat kita dengan server tersebut?
Untuk memudahkan kita, sekarang kita akan melabeli komputer di tempat kita dengan sebutan **local**
dan server tersebut dengan nama **bastion** dan berasumsi bahwa keduanya menjalankan sistem operasi
**Linux / Unix Based**.

Umumnya,  kita akan menggunakan SSH pada **local** untuk membuka koneksi dengan **bastion**. Tapi 
dalam kasus ini, hal ini tidak mungkin terjadi, karena kita tidak bisa secara langsung melakukan
SSH terhadap **bastion** ini.

Tapi bagaimana apabila ternyata konfigurasi jaringan menuju **local** ini cukup mudah dengan 
konfigurasi yang bisa diatur sendiri dan memperbolehkan akses SSH langsung menuju **local** ini?

Ya, maka jawabannya ada pada *Reverse SSH (tunneling)* yang disingkat menjadi **Reverse SSH**

Reverse SSH memperbolehkan kita untuk menggunakan jaringan yang sudah disediakan untuk melakukan
sebuah koneksi baru dari komputer lokal kita **local** menuju server **bastion**.

Karena awalnya koneksi jaringan ini berasal **DARI** si **bastion** menuju **local**, dan nantinya
kita akan menggunakan jaringan ini untuk mengakses si **bastion** dari **local** kita, sehingga
dinamakan **reverse**, dan karena SSH ini merupakan jaringan yang aman, sehingga kita akan 
melakukan koneksi yang aman *di atas* koneksi yang aman lagi.

Sehingga dinamakanlah *reverse SSH*

Skenario nya adalah sebagai berikut:
```
  1. Bastion ----> (ssh) ----> Local (Jaringan Terbentuk)
  2. Setelah jaringan terbentuk, akan ada -saluran- koneksi antara Bastion dengan Local
  3. Local ----> (ssh) -----> Bastion, untuk bisa masuk ke dalam Bastion dan mengeksekusi perintah
```

Salah satu syarat dari *reverse SSH* ini, adalah pada komputer **local** harus bisa diakses secara
*public*, dalam artian, harus memiliki *IP Address Public*.

Kemudian bagaimanakah apabila kita tidak memiliki *IP Address Public*, mis: pengguna komputer
rumahan di bawah ISP *pelat merah* ?

Mari kita coba lihat mengenai ngrok ini.

## Apa itu ngrok
ngrok merupakan suatu aplikasi *cross-platform* yang membuat seorang developer dapat mengekspos
sebuah server pada port tertentu di jaringan lokal, agar dapat digunakan global (internet) dengan
cara yang cukup sederhana.

Aplikasi di jaringan lokal, pada port tertentu, akan diekspose pada subdomain ngrok, pada port
tertentu juga, sehingga bisa diakses dari manapun.

sudah terdengar *menyerupai* kegunaan dari *IP Address Public* yang kita butuhkan bukan?

ngrok ini sendiri juga dapat digunakan untuk mem-*bypass* NAT dan *restriction* dari firewall dengan
membuat sebuah TCP tunnel dari sebuah subdomain acak pada ngrok.com.

Dan solusi aplikasi ngrok ini memliki layanan yang bersifat **g-r-a-t-i-s**.

Sehingga kita dapat menggunakan solusi ngrok ini dalam kasus yang disebutkan di atas.

## PoC & Showcase
Mari kita jabarkan lebih banyak mengenai kasus di atas

Komputer Lokal (**local**) 
- menggunakan sistem operasi Linux
- memiliki sebuah user bernama **withered-flowers** 
- memiliki password **only-for-testing-purpose**
- tidak memiliki *public ip* karena berada di bawah ISP *pelat merah*
- port ssh server yang digunakan adalah port default (**22**)
- port ssh yang dibuka agar local dapat mengakses bastion adalah **3000**

Komputer Server (**bastion**) 
- menggunakan sistem operasi Linux
- memiliki sebuah user bernama **student**
- mamiliki password **student**
- memiliki konfigurasi firewall yang cukup kompleks dan ip tidak diketahui

ngrok (**ngrok**)
- subdomain yang diberikan bernama **xxx.subdomain.ngrok.com**
- port yang diberikan adalah **54321**

### Konfigurasi Pertama - local
1. Kita akan melakukan instalasi ngrok terlebih dahulu dengan membuka situs ngrok
   [ngrok.com/download](https://ngrok.com/download) atau menggunakan package manager 
   yang disediakan distro linux yang digunakan
1. Melakukan registrasi pada situs ngrok [ngrok.com/signup](https://dashboard.ngrok.com/signup)
1. Mengambil *authorization token* yang diberikan pada situs ngrok tersebut.
   (mis: `x1234567890a`)
1. Membuka terminal pada **local**, dan memastikan bahwa ngrok sudah terpasang dengan mengetikkan
```shell
# tanda '>' merupakan penanda saja, jangan diketik
> ngrok
```
1. Registrasikan *authorization token* yang diberikan pada ngrok dengan mengetikkan 
   `ngrok authtoken <authtoken_yang_dicopy_dari_situs_ngrok_tadi>`, dengan contoh di atas maka yang
   diketikkan adalah
```shell
> ngrok authtoken x1234567890a
```
1. Pastikan service `sshd` pada **local** sudah berjalan
1. Expose port ssh (default adalah 22) ke public dengan menggunakan ngrok dengan mengetikkan
   `ngrok tcp <port_ssh_server_yang_ada_di_komputer_lokal>, dengan contoh sesuai di atas maka yang
   diketikkan adalah
```shell
> ngrok tcp 22
```
1. Setelah mengetik ini, ngrok akan mengekspose port 22 pada komputer local, mengikatnya dengan
   subdomain dan port yang diberikan oleh ngrok. dengan contoh sesuai di atas, maka
   port **22** di komputer local dapat diakses oleh luar melalui **xxx.subdomain.ngrok.com** pada
   port **54321**

### Konfigurasi Kedua - bastion
1. Login ke **bastion**
1. Buka terminal dari **bastion**
1. Menggunakan Reverse SSH dan membuka port tertentu agar dapat terkoneksi dengan **local** dengan
   cara 
   `ssh -R <port_yang_dibuka>:<ip_bastion>:<port_bastion> <akun_local>@<domain> -p <port>`, dengan
   contoh sesuai di atas maka yang diketikkan adalah
```shell
# buka port 3000 untuk tersambung ke bastion port 22
> ssh -R 3000:localhost:22 withered-flowers@xxx.subdomain.ngrok.io -p 54321
```
1. Setelah ini maka akan diminta untuk memasukkan password untuk user **withered-flowers** yaitu 
   **only-for-testing-purpose**
1. Sampai di sini, seharusnya **bastion** akan berhasil melakukan koneksi ssh terhadap **local**, 
   selanjutnya kita akan membuat agar **local** dapat menggunakan koneksi ssh yang sudah terjalin 
   ini agar dapat menggunakan ssh ke **bastion**

### Konfigurasi Ketiga - local melakukan ssh ke bastion
1. Setelah **bastion** berhasil melakukan koneksi ssh ke **local**, maka selanjutnya **local** yang
   akan melakukan koneksi ssh ke **bastion** dengan cara
   `ssh <akun_bastion>@<ip_local> -p <port_yang_dibuka_untuk_koneksi_remote>`, dengan contoh sesuai
   di atas maka yang diketikkan adalah
```shell
> ssh student@localhost -p 3000
```
1. Setelah ini maka akan diminta untuk memasukkan password untuk user **student** yaitu **student**
   juga.
1. Sampai di sini seharusnya komputer **local** berhasil melakukan akses ssh ke **bastion**.

Semoga berhasil yah !

## Referensi
- [Secure Shell](https://id.wikipedia.org/wiki/Secure_Shell)
- [Reverse SSH](https://www.howtogeek.com/428413/what-is-reverse-ssh-tunneling-and-how-to-use-it/)
- [ngrok What is](https://www.pubnub.com/learn/glossary/what-is-ngrok/)