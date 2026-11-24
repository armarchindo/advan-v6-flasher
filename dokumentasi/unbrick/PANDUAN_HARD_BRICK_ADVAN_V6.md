# 🛠️ Panduan Pemulihan Hard-Brick Modem Advan V6 (Unisoc Low-Level Dumper & Flasher)

![Advan V6 Unbrick Banner](assets/ADVAN-V6_brick.png)

Panduan ini merangkum penggunaan software flasher, driver USB, dan CLI utilities khusus untuk menangani proses **Low-Level Flashing, Dumping Partisi Flash, Ekstraksi ROM (.PAC), dan Unbrick Darurat (Hard-Brick / Mati Total / BROM Mode)** pada perangkat dengan chipset **Spreadtrum / Unisoc (khususnya Unisoc SL8563 Quad-Core ARMv7l pada modem Advan CPE V6)**.

> [!TIP]
> **Link Download Resmi Paket Driver & Script Recovery (.7z):**  
> Seluruh paket driver USB (`driver/`), engine flasher CLI (`spd_dump_tool/`), script recovery Python, dan launcher batch otomatis telah dipaketkan ke dalam arsip `.7z` yang dapat diunduh langsung pada rilis resmi GitHub berikut:  
> 📥 [**Download Driver & Script Recovery Advan V6 (.7z)**](https://github.com/armarchindo/advan-v6-flasher/releases/tag/driver_script)  
> *Setelah mengunduh, ekstrak file arsip `.7z` tersebut di PC Windows Anda untuk langsung menggunakannya.*

---

## 1. Ikhtisar & Struktur Direktori

Struktur folder paket dumper:
- **Folder Root:** Berisi folder driver, software pendukung, dan folder flasher.
- **Folder Flasher CLI (`spd_dump_tool/`):** Seluruh tool CLI eksekusi (`spd_dump.exe`, library, unpacker, helper) dikelompokkan menjadi satu kesatuan di folder ini.

```text
dumper/ (unisoc_dumper_tool/)
├── driver/                                      # Koleksi Driver USB Unisoc/Spreadtrum
│   ├── SPD_Driver_R4.20.4201/                   # Driver Resmi Rekomendasi (Win 7/8/10/11)
│   │   ├── Win10-Win11/ (DPInst64.exe, DPInst32.exe, sprdvcom.inf, sprdadb.inf)
│   │   ├── Win8/        (DPInst64.exe, DPInst32.exe, sprdvcom.inf, sprdadb.inf)
│   │   └── Win7/        (DPInst64.exe, DPInst32.exe, sprdvcom.inf, sprdadb.inf)
│   ├── SPRD_NPI_USB_Driver_v1.4/                # Driver Spreadtrum NPI Factory Mode
│   ├── Spreadtrum_Jungo_USB2Serial/             # Driver Jungo USB-to-Serial (Legacy)
│   └── Spreadtrum_SCI_USB2Serial/               # Driver SCI USB-to-Serial (Legacy)
└── spd_dump_tool/                               # Paket Utuh Flasher & Dumper CLI Unisoc
    ├── spd_dump.exe                             # Engine CLI Low-Level Flasher & Dumper Unisoc
    ├── libusb-1.0.dll                           # Library USB Driver User-space untuk spd_dump
    ├── unpac.exe                                # Extractor / Unpacker Firmware Resmi Unisoc (.PAC)
    ├── fphelper.exe                             # Flash Partition & Layout Helper Unisoc
    └── fphelper_t117.exe                        # Helper Khusus Chipset Unisoc T117
```

---

## 2. Penjelasan Fungsi Setiap File & Tool

### A. Core CLI Tools (Folder: `spd_dump_tool/`)

Seluruh tool CLI (`spd_dump.exe`, `libusb-1.0.dll`, `unpac.exe`, `fphelper.exe`, `fphelper_t117.exe`) berada dalam satu kesatuan folder `spd_dump_tool/` agar terisolasi rapi dan tidak tercecer di root direktori.

#### 1. `spd_dump.exe`
- **Fungsi Utama:**  
  Engine utama berbasis open-source libusb untuk berkomunikasi langsung dengan **BootROM silikon chip Unisoc (ROM code)** dan **FDL (Flash Downloader)**. Tool ini bekerja pada lapisan paling rendah (sebelum OS Linux atau U-Boot berjalan), sehingga dapat digunakan saat modem mengalami hardbrick/mati suri.
- **Kemampuan:**
  - Menghubungkan PC ke BootROM via USB handshaking (`checkbaud`).
  - Mengirim dan mengeksekusi bootloader sementara ke RAM: `FDL1` (SRAM) dan `FDL2` (DDR RAM).
  - Membaca (*dump/backup*) partisi NAND flash tertentu atau full flash memory.
  - Menulis ulang (*flashing/restore*) partisi NAND flash (misal partisi `ubipac`, `uboot`, `boot`).
  - Menghapus (*erase*) blok partisi NAND flash.
  - Membaca tabel partisi perangkat (`partition_list` / `partition_list_4k`).
  - Merestart (*reset/reboot*) modem secara hardware setelah selesai flashing.
- **Ketergantungan:** Wajib berada satu folder dengan `libusb-1.0.dll`.

#### 2. `libusb-1.0.dll`
- **Fungsi:**  
  Pustaka antarmuka USB mode pengguna (user-space C library) yang memungkinkan `spd_dump.exe` mendeteksi dan mengirim data raw USB transfer langsung ke Hardware ID Unisoc BootROM (`VID_1782 & PID_4D00` / `VID_1782 & PID_4045`).

#### 3. `unpac.exe`
- **Fungsi Utama:**  
  Tool untuk membongkar (*unpack/extract*) file arsip firmware resmi pabrikan Unisoc yang berekstensi `.pac` (misal update software resmi dari Advan atau Tozed).
- **Struktur File `.pac`:**  
  File `.pac` Unisoc adalah gabungan dari banyak file partisi dan bootloader, antara lain:
  - `FDL1` / `fdl1.bin` (First DownLoader - menginisialisasi clock & SRAM).
  - `FDL2` / `fdl2.bin` (Second DownLoader - menginisialisasi DDR RAM & controller flash SPI NAND).
  - `u-boot-spl-16k.bin` / `splloader.bin` (Secondary Program Loader).
  - `u-boot.bin` (U-Boot bootloader).
  - `boot.bin` (Kernel Linux zImage + Device Tree Blob).
  - `recovery.bin` (Kernel recovery).
  - `system.bin` / `ubipac.bin` (UBIFS System rootfs).
  - `prodnv.bin` (Konfigurasi kalibrasi RF, IMEI, SN pabrik).

#### 4. `fphelper.exe` & `fphelper_t117.exe`
- **Fungsi Utama:**  
  Tool pembantu (*Flash Partition Helper*) untuk menganalisis image hasil dump mentah (`flash.bin`).
- **Fitur:**
  - Mendeteksi ID chip Flash SPI NAND / NOR (`flash id`, layout, config).
  - Menghitung kapasitas memori flash (`guess: flash size = 256MB`).
  - Memeriksa header DHTB dan struktur signature firmware original.
  - `fphelper_t117.exe`: Varian khusus yang dioptimasi untuk arsitektur SoC feature-phone Unisoc T117.

---

### B. Direktori Driver (`driver/`)

Untuk menghubungkan modem Advan V6 ke PC melalui port Micro-USB, Windows memerlukan driver virtual COM port Unisoc yang sesuai. Tersedia 3 subfolder driver berdasarkan versi OS Windows Anda:

#### 1. `SPD_Driver_R4.20.4201` (**Rekomendasi Utama**)
* **Windows 10 & Windows 11:**  
  - **Lokasi Folder:** `driver\SPD_Driver_R4.20.4201\Win10-Win11\`  
  - **Installer:** Klik kanan `DPInst64.exe` (untuk 64-bit) atau `DPInst32.exe` (untuk 32-bit) $\rightarrow$ Run as Administrator.
* **Windows 8 & Windows 8.1:**  
  - **Lokasi Folder:** `driver\SPD_Driver_R4.20.4201\Win8\`  
  - **Installer:** Klik kanan `DPInst64.exe` (64-bit) atau `DPInst32.exe` (32-bit) $\rightarrow$ Run as Administrator.
* **Windows 7:**  
  - **Lokasi Folder:** `driver\SPD_Driver_R4.20.4201\Win7\`  
  - **Installer:** Klik kanan `DPInst64.exe` (64-bit) atau `DPInst32.exe` (32-bit) $\rightarrow$ Run as Administrator.

*Komponen Driver:*
- `sprdvcom.inf`: Driver Spreadtrum Virtual COM Port (menangani port BROM `SPRD U2S Diag`).
- `sprdadb.inf`: Driver Spreadtrum Android ADB Composite interface.
- `sprdvmdm.inf`: Driver Spreadtrum Modem & Diagnostic interface (`SPRD DIAG`, `SPRD AT`, `SPRD LOG`).
- `rdavcom.inf`: Driver RDA Microelectronics VCOM port.

#### 2. `SPRD_NPI_USB_Driver_v1.4`
- Paket driver Spreadtrum NPI (New Product Introduction) resmi untuk pengujian lini produksi, kalibrasi instrumen RF, dan pembacaan AT/DIAG port.

#### 3. `Spreadtrum_Jungo_USB2Serial` & `Spreadtrum_SCI_USB2Serial`
- Driver USB-to-Serial alternatif berbasis Jungo WinDriver dan SCI. Berguna sebagai driver cadangan jika sistem Windows menolak driver VCOM standar karena masalah digital signature.

---

## 3. Cara Memasukkan Modem ke Mode USB (Download / BROM Mode)

Sebelum menjalankan tool flasher di PC, modem harus dihubungkan ke PC via kabel Micro-USB data:

1. **Persiapan:**
   - Install driver terlebih dahulu sesuai versi Windows Anda:
     * **Win 10 / 11:** Jalankan `DPInst64.exe` di folder `driver\SPD_Driver_R4.20.4201\Win10-Win11\` sebagai Administrator.
     * **Win 8 / 8.1:** Jalankan `DPInst64.exe` di folder `driver\SPD_Driver_R4.20.4201\Win8\` sebagai Administrator.
     * **Win 7:** Jalankan `DPInst64.exe` di folder `driver\SPD_Driver_R4.20.4201\Win7\` sebagai Administrator.
   - Siapkan kabel data Micro-USB (pastikan kabel mendukung jalur DATA, bukan kabel power charging saja).
2. **Prosedur Hardware Trigger (WPS / Reset Method):**
   - Cabut adaptor listrik dari modem Advan V6 (modem dalam kondisi **MATI TOTAL**).
   - Colokkan kabel Micro-USB dari modem ke port USB laptop/PC.
   - **Tekan dan tahan tombol WPS** (atau tombol RESET) pada bodi modem, **JANGAN DILEPAS**.
   - Sambil tetap menahan tombol, **colokkan kabel power adaptor** ke modem.
   - Tetap tahan tombol selama **3–5 detik** sampai terdengar nada koneksi USB di Windows (*suara "tung-tung"*).
   - Lepaskan tombol WPS/RESET.
3. **Pengecekan di Device Manager Windows:**
   - **Kondisi A (Mode BROM / Download Murni):**  
     Muncul 1 port serial baru di bawah *Ports (COM & LPT)*:  
     `SPRD U2S Diag (COM...)` dengan Hardware ID `VID_1782&PID_4D00`. Ini adalah mode siap flash/dump tingkat paling rendah.
   - **Kondisi B (Mode Running Composite):**  
     Muncul satu set port lengkap:
     - `SPRD AT (COM...)`
     - `SPRD DIAG (COM...)`
     - `SPRD LOG (COM...)`  
     Mode ini menandakan modem dalam kondisi sistem aktif normal.

---

## 4. Panduan Praktis Penggunaan Tools

Semua perintah berikut dijalankan melalui **Command Prompt (CMD)** atau **PowerShell** dari dalam subfolder `spd_dump_tool`:

```powershell
cd spd_dump_tool
```

---

### Skenario 1: Membongkar Firmware Stock `.PAC` (`unpac.exe`)

Jika memiliki file ROM resmi pabrik berformat `.pac` (misalnya `firmware_stock.pac`):

1. **Melihat daftar partisi di dalam file PAC:**
   ```cmd
   unpac.exe list firmware_stock.pac
   ```
2. **Mengekstrak seluruh file partisi ke dalam folder `extracted_pac`:**
   ```cmd
   unpac.exe -d extracted_pac extract firmware_stock.pac
   ```
3. **Mengekstrak hanya partisi tertentu (misal mengambil FDL1 dan FDL2 untuk unbrick):**
   ```cmd
   unpac.exe -d fdl_files extract firmware_stock.pac FDL1 FDL2
   ```

---

### Skenario 2: Membaca Tabel Partisi Modem (`spd_dump.exe`)

Saat modem terdeteksi di mode BROM:

```cmd
spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 partition_list
```
*Output akan menampilkan daftar seluruh partisi NAND flash beserta ukuran dan sektornya.*

---

### Skenario 3: Membaca / Dump Backup Partisi Flash (`spd_dump.exe`)

1. **Membaca Partisi `prodnv` (Kalibrasi RF, SN, IMEI — Sangat Penting!):**
   ```cmd
   spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 read_part prodnv 0 0x800000 backup_prodnv.bin
   ```
2. **Membaca Partisi `ubipac` (Root filesystem sistem router):**
   ```cmd
   spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 read_part ubipac 0 0x8000000 backup_ubipac.bin
   ```
3. **Membaca Full Flash Chip (Dump Penuh 256MB):**
   ```cmd
   spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 read_flash 0 0x10000000 full_dump_256mb.bin
   ```

---

### Skenario 4: Memulihkan Modem Hardbrick (Flashing Partisi Rusak)

Jika router mengalami bootloop total, lampu mati, atau sistem corrupt:

1. Siapkan file image partisi yang sehat dari folder backup / dump partisi:
   - `mtd7_ubipac.bin` (Partisi UBI System)
   - `mtd1_uboot.bin` (Partisi U-Boot Bootloader)
   (Letakkan file image di folder `spd_dump_tool` atau tentukan path filenya).
2. Masukkan modem ke mode BROM (tahan WPS + colok power).
3. Jalankan perintah flashing partisi sistem:
   ```cmd
   spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 write_part ubipac mtd7_ubipac.bin reset
   ```
4. Modem akan otomatis ter-reflash dan melakukan restart (`reset`) kembali ke kondisi normal.

---

### Skenario 5: Analisis Hasil Dump Menggunakan `fphelper.exe`

Jika Anda memiliki file hasil dump mentah dan ingin mengetahui spesifikasi IC flash serta layout-nya:

```cmd
fphelper.exe full_dump_256mb.bin guess
```

Output akan menginfokan:
- Flash Manufacturer & Device ID (misal ESMT SPI NAND / Winbond / Dosilicon).
- Kapasitas chip (256MB / 2048Mbit).
- Blok alokasi DHTB signature.

---

## 5. Ringkasan Cepat Cheat-Sheet Perintah

| Kebutuhan / Tugas | Tool | Sintaks Perintah |
|---|---|---|
| **Pindah ke Direktori Tool** | Shell | `cd spd_dump_tool` |
| **Ekstrak File .PAC** | `unpac.exe` | `unpac.exe -d output/ extract firmware.pac` |
| **Cek Koneksi BROM** | `spd_dump.exe` | `spd_dump.exe checkbaud 115200` |
| **Lihat Partisi Modem** | `spd_dump.exe` | `spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 partition_list` |
| **Dump Partisi Tertentu** | `spd_dump.exe` | `spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 read_part <nama_partisi> 0 <ukuran> <output.bin>` |
| **Flash Partisi Tertentu** | `spd_dump.exe` | `spd_dump.exe fdl fdl1.bin 0x40004000 fdl fdl2.bin 0x40008000 write_part <nama_partisi> <input.bin> reset` |
| **Tebak Layout Flash** | `fphelper.exe` | `fphelper.exe dump.bin guess` |
| **Install Driver Win 7 / 8 / 10 / 11** | Driver | Buka folder `driver\SPD_Driver_R4.20.4201\<Versi_OS>\` $\rightarrow$ Klik kanan `DPInst64.exe` $\rightarrow$ Run as Admin |
| **Download Driver & Script (.7z)** | Browser | [GitHub Release: driver_script](https://github.com/armarchindo/advan-v6-flasher/releases/tag/driver_script) |
