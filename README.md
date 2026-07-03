# Resilient School Network Infrastructure: Multi-Segment DHCP Allocation with Telegram Alerting

Proyek ini mendemonstrasikan implementasi arsitektur jaringan sekolah yang andal dan terfragmentasi menggunakan **MikroTik RouterOS (CHR)** di dalam simulator **GNS3**. Infrastruktur ini menerapkan alokasi DHCP berbasis port fisik yang diintegrasikan dengan sistem pemantauan proaktif berbasis kejadian (*event-driven*). Sistem akan mengirimkan notifikasi telemetri secara *real-time* melalui **Telegram Bot API** setiap kali terjadi perubahan status pada tautan jaringan (*link state*).

---

## 🛠️ Arsitektur & Segmentasi Jaringan
Jaringan ini dibagi menjadi tiga zona fungsional utama secara logis dan fisik untuk mencegah konflik IP, mengoptimalkan *broadcast domain*, serta mempermudah proses pencarian sumber masalah (*troubleshooting*):
* **Segmen Guru (`ether1`):** Network `192.168.10.0/24` (Alokasi IP Dinamis untuk PC Guru).
* **Segmen Lab (`ether3`):** Network `192.168.20.0/24` (Alokasi IP Dinamis untuk workstation Lab).
* **Segmen Siswa & Server (`bridge-siswa`):** Network `192.168.30.0/24` (Menggabungkan `ether4` dan `ether5` untuk menyatukan PC Siswa dan Server Sekolah kritis dalam satu segmen operasional).

<img width="662" height="331" alt="workspace" src="https://github.com/user-attachments/assets/4d4e7a6f-a0d3-471c-8f2b-98cafd7492d8" />


---

## ⚡ Konfigurasi Pemantauan & Otomatisasi Telemetri
Sistem memanfaatkan fitur bawaan MikroTik **Netwatch** untuk melakukan *ICMP Echo Request (Ping)* setiap 5 detik ke perangkat-perangkat kritis. Begitu mendeteksi perubahan status (`UP` atau `DOWN`), switch MikroTik akan mengeksekusi perintah *HTTP POST Request* yang aman langsung ke API Server Telegram.

<img width="607" height="262" alt="Screenshot 2026-07-02 182708" src="https://github.com/user-attachments/assets/99d6cd47-a5eb-4b54-ab5a-06464e01cee9" />


### Perintah Script di Terminal Winbox:
*Arahkan kursor ke pojok kanan atas setiap kotak kode di bawah ini dan klik tombol **Copy** untuk menyalin script.*

### 1. Script Saat Jalur Putus / Mati (DOWN State)
Gunakan script ini di terminal atau masukkan ke tab **Down** pada Netwatch untuk memberikan alarm saat server atau kabel kecabut:
```
/tool fetch http-method=post url="[https://api.telegram.org/bot](https://api.telegram.org/bot)<TOKEN_BOT_ANDA>/sendMessage" http-data="chat_id=<ID_CHAT_ANDA>&text=⚠️ ALERT: Jalur Infrastruktur Jaringan DOWN / Terputus! Segera lakukan pengecekan." keep-result=no

```
2. Script Saat Jalur Kembali Normal (UP State)
Gunakan script ini di terminal atau masukkan ke tab Up pada Netwatch ketika server dicolok kembali dan sistem kembali pulih:

```
/tool fetch http-method=post url="[https://api.telegram.org/bot](https://api.telegram.org/bot)<TOKEN_BOT_ANDA>/sendMessage" http-data="chat_id=<ID_CHAT_ANDA>&text=✅ INFO: Jalur Infrastruktur Jaringan UP / Berhasil Terhubung Kembali." keep-result=no
```
🚀 Langkah-Langkah Implementasi
Langkah 1: Setup Bot Telegram
Buka Telegram, cari akun @BotFather, buat bot baru, lalu simpan kode Bot Token yang diberikan.

Dapatkan ID akun Telegram pribadi Anda menggunakan bot pencari ID seperti
```
@userinfobot.
```
Validasi awal apakah bot dan ID Anda sudah sinkron dengan mengetik langsung di terminal Winbox menggunakan perintah uji coba berikut:

```
/tool fetch http-method=post url="[https://api.telegram.org/bot](https://api.telegram.org/bot)<TOKEN_BOT_LU>/sendMessage" http-data="chat_id=<ID_CHAT_ANDA>&text=TesKoneksiSistem" keep-result=no
```
Langkah 2: Konfigurasi Netwatch pada Winbox
Masuk ke MikroTik Switch menggunakan aplikasi Winbox.

Buka menu Tools > Netwatch, lalu klik tanda + (Tambah Baru).

Pada tab Host, masukkan IP Address perangkat target yang ingin diawasi (misal IP Server Sekolah) dan atur Interval menjadi 00:00:05 (5 detik).

Masuk ke tab Up, salin Script UP State di atas, lalu paste ke dalam kolom. Jangan lupa ganti <TOKEN_BOT_ANDA> dan <ID_CHAT_ANDA> dengan data asli milik Anda.

Masuk ke tab Down, salin Script DOWN State di atas, lalu paste ke dalam kolom. Sesuaikan kembali token dan ID chat Anda.

Klik Apply dan OK.

📈 Validasi & Pembuktian Projek (Proof of Concept)
Verifikasi DHCP: Setiap perangkat klien (VPCS) yang dicolokkan ke port fisik yang sesuai terbukti sukses melakukan proses jabat tangan jaringan (DHCP DORA) secara polosan dan mendapatkan status Bound.

<img width="447" height="170" alt="Screenshot 2026-07-02 182540" src="https://github.com/user-attachments/assets/38ceb5ba-78db-4cfd-acda-1704c70ae29c" />
<br><br>

Simulasi Gangguan Jaringan: Ketika kabel salah satu perangkat dicabut di dalam simulator GNS3, fitur Netwatch langsung mendeteksi perubahan status dari up menjadi down dalam rentang waktu 5 detik, dan seketika itu juga Switch berhasil memerintahkan bot Telegram untuk mengirimkan pesan peringatan bahaya langsung ke HP administrator jaringan.

<img width="972" height="865" alt="Screenshot 2026-07-02 182919" src="https://github.com/user-attachments/assets/4fedf115-f9d2-43c5-9975-c0fb93e109cc" />


📝 Kesimpulan
Keamanan Optimal: Segmentasi port fisik berhasil mengisolasi trafik antar divisi sekolah dan mencegah konflik IP secara total.

Respon Instan (MTTR): Otomatisasi Netwatch dan Telegram API memangkas waktu penanganan gangguan jaringan secara real-time.

Efisiensi Tinggi: Penggunaan metode HTTP POST via internal script terbukti ringan dan andal tanpa membebani performa perangkat router.
