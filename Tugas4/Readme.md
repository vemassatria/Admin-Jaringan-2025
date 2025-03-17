<h1 align="center">
  "Konfigurasi DNS untuk Jaringan Internal"
</h1>

# Konfigurasi DNS untuk Jaringan Internal

## 1. Mengecek Status BIND

![bindstatus](/Tugas_DNS/img/bindstatus.png)

```sh
systemctl status bind9
```
**Penjelasan:**
Perintah ini digunakan untuk memastikan apakah layanan BIND sedang berjalan atau tidak.

## 2. Melakukan Restart BIND untuk Menerapkan Perubahan
```sh
systemctl restart bind9
```
**Penjelasan:**
Jika ada perubahan dalam konfigurasi DNS, layanan BIND perlu di-restart agar perubahan tersebut diterapkan.

## 3. Mengonfigurasi Zona DNS dalam `named.conf.local`

![namedconflocal](/Tugas_DNS/img/namedconflocal.png)

```sh
nano /etc/bind/named.conf.local
```
Tambahkan konfigurasi berikut:

```sh
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};

zone "100.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

**Penjelasan:**
Konfigurasi ini mendeklarasikan zona forward lookup dan reverse lookup untuk jaringan internal.

## 4. Membuat File Forward Zone

![dbexample](/Tugas_DNS/img/dbexample.png)

```sh
nano /etc/bind/db.example.com
```
Isi file:

```sh
$TTL 86400
@   IN  SOA  ns1.example.com. root.example.com. (
            2024031501
            3600
            1800
            604800
            86400 )

@   IN  NS  ns1.example.com.
ns1 IN  A   192.168.100.57
www IN  A   192.168.100.57
```

**Penjelasan:**
File ini mendefinisikan alamat IP untuk `ns1` dan `www` dalam jaringan internal.

## 5. Membuat File Reverse Zone

![db192](/Tugas_DNS/img/dbrev.png)

```sh
nano /etc/bind/db.192
```
Isi file:

```sh
$TTL 86400
@   IN  SOA  ns1.example.com. root.example.com. (
            2024031501
            3600
            1800
            604800
            86400 )

@   IN  NS  ns1.example.com.
57  IN  PTR ns1.example.com.
57  IN  PTR www.example.com.
```

**Penjelasan:**
File ini memungkinkan resolusi alamat IP ke nama domain dalam reverse DNS lookup.

## 6. Mengecek Konfigurasi

![testconf](/Tugas_DNS/img/testconf.png)

```sh
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/db.example.com
sudo named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192
```

**Penjelasan:**
Perintah ini digunakan untuk memeriksa apakah konfigurasi BIND telah benar sebelum dijalankan.

## 7. Menguji Query Forward

![testforward](/Tugas_DNS/img/test.png)

```sh
dig example.com @192.168.100.57
```

**Penjelasan:**
Perintah ini memverifikasi apakah DNS dapat mengonversi domain `example.com` ke alamat IP dengan benar.

## 8. Menguji Query Reverse

![testreversed](/Tugas_DNS/img/testrev.png)

```sh
dig -x 192.168.100.57 @192.168.100.57
```

**Penjelasan:**
Perintah ini memastikan bahwa alamat IP dapat dikembalikan menjadi nama domain dengan benar.

## Kesimpulan

Konfigurasi ini memastikan bahwa BIND berfungsi dengan baik sebagai DNS internal. Jika terjadi kendala, periksa kembali izin akses, firewall, serta log sistem menggunakan `journalctl -xe | grep named`.

