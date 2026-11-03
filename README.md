# Firewall-dan-Network-Monitoring-Lab

## Project Overview

Project ini merupakan praktikum konfigurasi firewall dan monitoring jaringan menggunakan **pfSense** dan **Kali Linux** pada VirtualBox.

Tujuan proyek ini adalah memahami cara kerja firewall dalam mengontrol lalu lintas jaringan, menerapkan aturan pemblokiran HTTP, menguji koneksi HTTP dan HTTPS, serta memantau aktivitas jaringan melalui firewall logs.

## Tools and Environment

| Komponen | Keterangan |
|---|---|
| Host OS | Windows 11 |
| Virtualization | Oracle VirtualBox |
| Firewall | pfSense CE 2.9.0 |
| Testing Machine | Kali Linux 2025.3 |
| Network Type | NAT, Host-Only, Internal Network |

## Network Topology

Topologi jaringan yang digunakan:

```text
                 Internet
                    |
               NAT (WAN)
                    |
              +-----------+
              |  pfSense  |
              | Firewall  |
              +-----------+
                |       |
          LAN em2     OPT1 em1
       192.168.10.1  192.168.56.2
                |       |
          Internal    Host-Only
          Network     Network
                |       |
           Kali Linux  Windows 11
         192.168.10.100 192.168.56.1
```

### IP Address Configuration

| Perangkat | Interface | IP Address |
|---|---|---|
| pfSense | WAN (em0) | 10.0.2.15 |
| pfSense | LAN (em2) | 192.168.10.1/24 |
| pfSense | OPT1 (em1) | 192.168.56.2/24 |
| Windows 11 | Host-Only | 192.168.56.1/24 |
| Kali Linux | LAN | 192.168.10.100/24 |

Kali Linux memperoleh IP address melalui DHCP dari pfSense.

## Configuration Steps

### A. pfSense Installation

1. Membuat virtual machine pfSense di VirtualBox.
2. Mengatur Adapter 1 sebagai NAT untuk koneksi WAN.
3. Mengatur Adapter 2 sebagai Host-Only untuk akses administrasi dari Windows.
4. Mengatur Adapter 3 sebagai Internal Network dengan nama `P5-LAN`.
5. Melakukan instalasi pfSense dan konfigurasi interface.

### B. Interface Configuration

Interface pfSense yang digunakan:

- WAN: `em0`
- LAN: `em2`
- OPT1: `em1`

Alamat IP LAN dikonfigurasi menjadi `192.168.10.1/24`, sedangkan OPT1 menggunakan `192.168.56.2/24`.

OPT1 digunakan agar Windows 11 tetap dapat mengakses WebConfigurator pfSense melalui jaringan Host-Only.

### C. DHCP Configuration

DHCP server diaktifkan pada interface LAN dengan konfigurasi:

- DHCP Range: `192.168.10.100 – 192.168.10.199`
- Subnet: `192.168.10.0/24`
- Gateway: `192.168.10.1`

Kali Linux berhasil memperoleh IP address `192.168.10.100`.

### D. Firewall Rule Configuration

Dibuat firewall rule pada interface LAN untuk memblokir koneksi HTTP keluar.

| Parameter | Konfigurasi |
|---|---|
| Action | Block |
| Interface | LAN |
| Address Family | IPv4 |
| Protocol | TCP |
| Source | LAN subnets |
| Destination | Any |
| Destination Port | HTTP (80) |
| Description | LAB - Block HTTP outbound |

Rule ditempatkan di atas aturan default allow LAN agar koneksi HTTP diblokir sebelum mencapai aturan yang mengizinkan lalu lintas lainnya.

## Testing and Results

Pengujian dilakukan menggunakan terminal Kali Linux.

### A. HTTP Connection Test

Command:

```bash
curl -I --connect-timeout 5 -4 http://neverssl.com
```

Hasil:

```text
curl: (28) Connection timed out
```

**Analisis:** Koneksi HTTP mengalami timeout. Hasil ini sesuai dengan tujuan firewall rule untuk memblokir lalu lintas TCP port 80.

### B. HTTPS Connection Test

Command:

```bash
curl -I --max-time 10 https://example.com
```

Hasil:

```text
HTTP/2 200
```

**Analisis:** Koneksi HTTPS berhasil. Hal ini menunjukkan bahwa koneksi melalui port 443 tetap dapat dilakukan.

### C. Firewall Log Monitoring

Monitoring dilakukan melalui menu:

`Status → System Logs → Firewall`

Log menunjukkan adanya aktivitas yang diblokir oleh rule:

`LAB - Block HTTP outbound`

Contoh lalu lintas yang tercatat:

- Source: `192.168.10.100`
- Destination: `1.1.1.1`
- Destination Port: `80`
- Protocol: TCP
- Action: Block

Log tersebut menunjukkan bahwa firewall mendeteksi dan memblokir percobaan koneksi HTTP dari Kali Linux.

## Screenshots

### 1. pfSense Dashboard
![pfSense Dashboard](screenshots/01-pfsense-dashboard.png)

### 2. Network Configuration
![Network Configuration](screenshots/02-network-configuration.png)

### 3. Firewall Rule LAN
![Firewall Rule](screenshots/03-firewall-rule-lan.png)

### 4. HTTP Blocked
![HTTP Blocked](screenshots/04-kali-http-blocked.png)

### 5. HTTPS Allowed
![HTTPS Allowed](screenshots/05-kali-https-allowed.png)

### 6. Firewall Logs
![Firewall Logs](screenshots/06-firewall-logs.png)

## Conclusion

Berdasarkan praktikum yang telah dilakukan, pfSense berhasil dikonfigurasi sebagai firewall untuk mengontrol lalu lintas jaringan antara Kali Linux dan internet.

Firewall rule berhasil memblokir koneksi HTTP melalui TCP port 80, sementara koneksi HTTPS melalui port 443 tetap berhasil. Aktivitas pemblokiran juga dapat diamati melalui Firewall Logs pada pfSense.

Praktikum ini memberikan pemahaman mengenai konfigurasi interface jaringan, DHCP, firewall rules, pengujian konektivitas, dan monitoring aktivitas jaringan menggunakan pfSense.

---

**Project:** Project 5 — Firewall dan Network Monitoring Lab  
**Tools:** pfSense, Kali Linux, Oracle VirtualBox  
**Platform:** Windows 11
