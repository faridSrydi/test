# Q-GUARD: PANDUAN WIRING HARDWARE LENGKAP & DETAIL (KABEL-DEMI-KABEL)

Panduan ini dibuat secara **sangat detail, langkah demi langkah, kabel demi kabel (pin-ke-pin)**. 
Bahkan jika kamu baru pertama kali merakit IoT, ikuti panduan ini urut dari Bagian 1 sampai Bagian 8.

---

## DAFTAR ISI
1. [Prinsip Jalur Daya (Breadboard Rail)](#1-prinsip-jalur-daya-breadboard-rail)
2. [Modul Daya: Adaptor USB, TP4056, 2x 18650, Saklar, & MT3608](#2-modul-daya-adaptor-usb-tp4056-2x-18650-saklar--mt3608)
3. [TFT LCD 2.8" ILI9341 + Touchscreen XPT2046 (Pin-demi-Pin)](#3-tft-lcd-28-ili9341--touchscreen-xpt2046-pin-demi-pin)
4. [Modul MicroSD Card Adapter (SPI)](#4-modul-microsd-card-adapter-spi)
5. [Tiga Sensor I2C: BME280, MPU6050, & DS3231 RTC](#5-tiga-sensor-i2c-bme280-mpu6050--ds3231-rtc)
6. [Tiga Sensor Suhu Probe DS18B20 (T1, T2, T3)](#6-tiga-sensor-suhu-probe-ds18b20-t1-t2-t3)
7. [Modul GPS NEO-6M (UART2)](#7-modul-gps-neo-6m-uart2)
8. [Tiga Tombol 3-Pin, Sensor Pintu MC-38, 3 LED, & Buzzer 5V](#8-tiga-tombol-3-pin-sensor-pintu-mc-38-3-led--buzzer-5v)
9. [Master Pinout Map ESP32 (Tabel Rangkuman Total)](#9-master-pinout-map-esp32-tabel-rangkuman-total)

---

## 1. PRINSIP JALUR DAYA (BREADBOARD RAIL)

Karena kamu memiliki banyak sensor, kita membutuhkan **3 Jalur Rel Utama** pada breadboard / PCB:

```
[ Rel 5.0V ] ═══════════════════════════════════════ (Kabel MERAH dari MT3608 VOUT+)
[ Rel 3.3V ] ═══════════════════════════════════════ (Kabel ORANYE dari ESP32 Pin 3V3)
[ Rel GND  ] ═══════════════════════════════════════ (Kabel HITAM - Ground Bersama)
```

* **Rel 5.0V:** Menyuplai daya ke pin **VIN ESP32**, **GPS NEO-6M**, **MicroSD Module**, dan **Active Buzzer 5V**.
* **Rel 3.3V:** Dihasilkan dari pin **3V3 ESP32** untuk menyuplai logika aman ke **BME280**, **MPU6050**, **DS3231 RTC**, **3x DS18B20**, **TFT LCD**, dan **3 Tombol**.
* **Rel GND (Wajib Satu Ground):** SEMUA kutub negatif (-) modul, baterai, adaptor, dan sensor **wajib terhubung bersama** ke rel ini agar sinyal tidak error.

---

## 2. MODUL DAYA: ADAPTOR USB, TP4056, 2x 18650, SAKLAR, & MT3608

Modul ini bertugas menerima colokan listrik adaptor 5V 3A, mengisi baterai, dan menghasilkan 5.0V stabil ke ESP32.

```
                    [ 2x Baterai 18650 Paralel ]
                      (+) Kutub Positif Bersama
                      (-) Kutub Negatif Bersama
                               |       |
                               |       |
      Adaptor 5V 3A            v       v
    (Colokan Type-C/Micro)   +---+   +---+
            |                |B+ |   |B- |
            v                +---+   +---+
    +----------------------------------------------+
    │          MODUL CHARGER TP4056                │
    │   (Ada chip proteksi DW01A + 8205A)          │
    │                                              │
    │     OUT+                          OUT-       │
    +------|-----------------------------|---------+
           |                             |
     [SAKLAR ON/OFF]                     |
      (Kaki 1 ke OUT+)                   |
      (Kaki 2 lanjut)                    |
           |                             |
           v                             v
       +-------+                     +-------+
       │ VIN+  │                     │ VIN-  │
       +-------+                     +-------+
    +----------------------------------------------+
    │          MODUL BOOST MT3608                  │
    │  [Putar baut potensiometer ke 5.0V!]         │
    │                                              │
    │     VOUT+                         VOUT-      │
    +------|-----------------------------|---------+
           |                             |
           v                             v
      Rel 5.0V                      Rel GND
   (Ke ESP32 VIN)                (Ke ESP32 GND)
```

### Sambungan Kabel Modul Daya:
1. **Dua Baterai 18650 Paralel:**
   - Hubungkan kutub positif (+) Baterai 1 ke kutub (+) Baterai 2 -> Solder ke pin **B+** TP4056.
   - Hubungkan kutub negatif (-) Baterai 1 ke kutub (-) Baterai 2 -> Solder ke pin **B-** TP4056.
   - *Tegangan tetap nominal 3.7V (maksimal 4.2V saat penuh), kapasitas baterai berlipat ganda (~4000-6000 mAh).*
2. **Kabel Saklar ON/OFF:**
   - Sambungkan pin **OUT+** TP4056 -> ke **Kaki 1 Saklar**.
   - Sambungkan **Kaki 2 Saklar** -> ke pin **VIN+** MT3608.
   - Sambungkan pin **OUT-** TP4056 -> langsung ke pin **VIN-** MT3608.
3. **Penyetelan MT3608 (JANGAN DILEWATKAN):**
   - Hidupkan Saklar ke posisi **ON**.
   - Tempelkan jarum tester Multimeter (Mode DC Volt): merah ke `VOUT+`, hitam ke `VOUT-`.
   - Putar obeng minus pada baut kuningan MT3608 berlawanan arah jarum jam sampai layar multimeter menunjukkan angka **5.00V**.
   - Matikan saklar (OFF).
4. **Sambungan ke ESP32:**
   - Sambungkan **VOUT+ MT3608** -> ke **Rel 5.0V** breadboard -> colok ke pin **VIN ESP32**.
   - Sambungkan **VOUT- MT3608** -> ke **Rel GND** breadboard -> colok ke pin **GND ESP32**.

---

## 3. TFT LCD 2.8" ILI9341 + TOUCHSCREEN XPT2046 (PIN-DEMI-PIN)

Modul layar 2.8" memiliki deretan 14 pin (atau 18 pin jika ada slot SD bawaan). Kita gunakan pin layar dan sentuhnya:

```
  +---------------------------------------------------------+
  |                 LAYAR TFT 2.8" ILI9341                  |
  |                                                         |
  | [VCC] [GND] [CS] [RESET] [DC] [MOSI] [SCK] [LED] [MISO] |
  |                                                         |
  |          [T_CLK] [T_CS] [T_DIN] [T_DO] [T_IRQ]          |
  +---------------------------------------------------------+
```

### Tabel Sambungan Layar & Touch ke ESP32:

| Pin pada Layar TFT | Nama Pin | Hubungkan ke Mana? | Keterangan |
|:---|:---|:---|:---|
| **Pin 1** | **VCC** | **Rel 3.3V** | Daya logika layar |
| **Pin 2** | **GND** | **Rel GND** | Ground |
| **Pin 3** | **CS** | **GPIO 5** | Chip Select LCD |
| **Pin 4** | **RESET** | **GPIO 26** | Reset Hardware LCD |
| **Pin 5** | **DC / RS**| **GPIO 27** | Data/Command Selection |
| **Pin 6** | **SDI / MOSI**| **GPIO 23** | SPI Master Out (Jalur Bersama) |
| **Pin 7** | **SCK / CLK** | **GPIO 18** | SPI Clock (Jalur Bersama) |
| **Pin 8** | **LED / BL** | **Rel 3.3V** | Lampu latar layar (Backlight ON) |
| **Pin 9** | **SDO / MISO**| **GPIO 19** | SPI Master In (Jalur Bersama) |
| **Pin 10** | **T_CLK** | **GPIO 18** | Clock Touchscreen (Jalur bersama SCK) |
| **Pin 11** | **T_CS** | **GPIO 14** | Chip Select Touchscreen |
| **Pin 12** | **T_DIN** | **GPIO 23** | Input Data Touch (Jalur bersama MOSI) |
| **Pin 13** | **T_DO** | **GPIO 19** | Output Data Touch (Jalur bersama MISO) |
| **Pin 14** | **T_IRQ** | *(KOSONGKAN)* | Tidak dipakai (Touch dibaca via polling) |

---

## 4. MODUL MICROSD CARD ADAPTER (SPI)

Modul MicroSD mandiri memiliki 6 pin:

| Pin Modul MicroSD | Hubungkan ke Mana? | Keterangan |
|:---|:---|:---|
| **CS** | **GPIO 13** | Chip Select MicroSD |
| **SCK** | **GPIO 18** | SPI Clock (Gabung ke jalur bersama SCK) |
| **MOSI** | **GPIO 23** | SPI MOSI (Gabung ke jalur bersama MOSI) |
| **MISO** | **GPIO 19** | SPI MISO (Gabung ke jalur bersama MISO) |
| **VCC** | **Rel 5.0V (VIN)** | Modul MicroSD butuh 5V karena ada regulator LDO bawaan |
| **GND** | **Rel GND** | Ground |

---

## 5. TIGA SENSOR I2C: BME280, MPU6050, & DS3231 RTC

Semua sensor I2C memakai 2 kabel data yang sama (**GPIO 21** untuk Data SDA, dan **GPIO 22** untuk Clock SCL).

### A. Modul BME280 (Suhu & Kelembapan Luar):
* `VCC` -> **Rel 3.3V**
* `GND` -> **Rel GND**
* `SDA` -> **GPIO 21**
* `SCL` -> **GPIO 22**

### B. Modul MPU6050 GY-521 (Akselerometer/Guncangan):
* `VCC` -> **Rel 3.3V**
* `GND` -> **Rel GND**
* `SDA` -> **GPIO 21**
* `SCL` -> **GPIO 22**
* **PIN AD0 (SANGAT PENTING):** Sambungkan pin **AD0 ke Rel 3.3V** menggunakan seutas kabel kecil.
  *(Tujuannya: mengubah alamat I2C MPU6050 menjadi `0x69` agar tidak tabrakan dengan DS3231).*

### C. Modul RTC DS3231 (Jam Presisi Real-Time):
* `VCC` -> **Rel 3.3V**
* `GND` -> **Rel GND**
* `SDA` -> **GPIO 21**
* `SCL` -> **GPIO 22**
* *(Pasang baterai koin CR2032 pada modul DS3231).*

---

## 6. TIGA SENSOR SUHU PROBE DS18B20 (T1, T2, T3)

Ketiga sensor probe memiliki 3 kabel (Merah, Hitam, Kuning/Putih). Sambungkan ketiga probe secara paralel:

```
                    Kabel MERAH Probe 1 ----+
                    Kabel MERAH Probe 2 ----+---> Ke Rel 3.3V
                    Kabel MERAH Probe 3 ----+

                    Kabel HITAM Probe 1 ----+
                    Kabel HITAM Probe 2 ----+---> Ke Rel GND
                    Kabel HITAM Probe 3 ----+

                    Kabel KUNING Probe 1 ---+
                    Kabel KUNING Probe 2 ---+---> Ke ESP32 GPIO 4
                    Kabel KUNING Probe 3 ---+
                                            |
                                            v
                           [ Resistor 4.7 kOhm Pull-up ]
                                            |
                                            v
                                       Ke Rel 3.3V
```

### Langkah Pasang:
1. Satukan 3 kabel Merah -> tancapkan ke **Rel 3.3V**.
2. Satukan 3 kabel Hitam -> tancapkan ke **Rel GND**.
3. Satukan 3 kabel Kuning -> tancapkan ke lubang **GPIO 4**.
4. Pasang **1 buah Resistor 4.7 kOhm**: satu kakinya colok ke **GPIO 4** (gabung kabel kuning), satu kakinya lagi colok ke **Rel 3.3V**.

---

## 7. MODUL GPS NEO-6M (GY-GPS6MV2)

Modul GPS memiliki 4 pin:

| Pin GPS NEO-6M | Hubungkan ke Mana? | Keterangan |
|:---|:---|:---|
| **VCC** | **Rel 5.0V (VIN)** | Diberi 5V agar penangkapan sinyal satelit kuat |
| **GND** | **Rel GND** | Ground |
| **TX** | **GPIO 16 (RX2)** | TX GPS mengirim data ke RX2 ESP32 |
| **RX** | **GPIO 17 (TX2)** | RX GPS menerima perintah dari TX2 ESP32 |

*(Antena keramik kotak dipasang mengarah ke atas/langit).*

---

## 8. TIGA TOMBOL 3-PIN, SENSOR PINTU MC-38, 3 LED, & BUZZER 5V

### A. Tiga Modul Tombol Tactile (3-Pin: VCC, GND, OUT):
Setiap modul tombol memiliki 3 kaki bertuliskan `VCC`, `GND`, dan `OUT` (atau `S`).

```
Modul Tombol 3-Pin:
  [ VCC ] ---------> Ke Rel 3.3V  (JANGAN KE 5V!)
  [ GND ] ---------> Ke Rel GND
  [ OUT ] ---------> Ke Pin GPIO ESP32
```

1. **Tombol 1 (BACK):**
   - `VCC` -> **Rel 3.3V**
   - `GND` -> **Rel GND**
   - `OUT` -> **GPIO 34**
2. **Tombol 2 (SELECT / GESER MENU):**
   - `VCC` -> **Rel 3.3V**
   - `GND` -> **Rel GND**
   - `OUT` -> **GPIO 36 (VP)**
3. **Tombol 3 (CONFIRM / ENTER / OK):**
   - `VCC` -> **Rel 3.3V**
   - `GND` -> **Rel GND**
   - `OUT` -> **GPIO 39 (VN)**

---

### B. Sensor Pintu Magnetik MC-38 (2 Kabel):
Sensor sakelar magnetik ini tidak memiliki polaritas (+/- bebas):
1. **Kabel 1:** Sambungkan ke **GPIO 35**.
2. **Kabel 2:** Sambungkan ke **Rel GND**.
3. **Resistor Pull-up (4.7 kOhm):** Pasang resistor 4.7 kOhm di antara **GPIO 35** dan **Rel 3.3V**.

---

### C. Tiga Buah LED Indikator 5mm:
Setiap LED memiliki 2 kaki: Kaki Panjang = **Anoda (+)**, Kaki Pendek = **Katoda (-)**.

1. **LED Hijau (Status Aman / Normal):**
   - Kaki Panjang (+) -> Pasang **Resistor 330 Ohm** -> sambung ke **GPIO 32**
   - Kaki Pendek (-) -> Sambung ke **Rel GND**
2. **LED Kuning (Peringatan / Caution):**
   - Kaki Panjang (+) -> Pasang **Resistor 330 Ohm** -> sambung ke **GPIO 33**
   - Kaki Pendek (-) -> Sambung ke **Rel GND**
3. **LED Merah (Bahaya / Excursion):**
   - Kaki Panjang (+) -> Pasang **Resistor 330 Ohm** -> sambung ke **GPIO 25**
   - Kaki Pendek (-) -> Sambung ke **Rel GND**

*(Jika stok resistor 330 Ohm habis, kamu bisa pakai resistor 220 Ohm).*

---

### D. Active Buzzer 5V:

1. **Jika Buzzer kamu tipe MODUL 3-Pin (VCC, GND, I/O):**
   - `VCC` -> **Rel 5.0V (VIN)**
   - `GND` -> **Rel GND**
   - `I/O` -> **GPIO 12**
   - Pasang resistor **1 kOhm** di antara **GPIO 12 dan GND** (Pull-down pengaman boot).

2. **Jika Buzzer kamu tipe TABUNG 2-Pin Hitam (+ dan - biasa):**
   - Kaki Panjang bertanda (+) -> Sambung ke **Rel 5.0V (VIN)**
   - Kaki Pendek (-) -> Sambung ke **GPIO 12**
   - Pasang resistor **1 kOhm** di antara **GPIO 12 dan GND** (Pull-down pengaman boot).

> **KENAPA PERLU RESISTOR 1 kOhm PADA GPIO 12?**
> Pin GPIO 12 pada ESP32 adalah pin *boot strapping* (MTDI). Jika pin ini melayang atau berlogika HIGH saat alat dinyalakan, ESP32 akan salah membaca voltase flash internal dan langsung crash / bootloop. Resistor 1 kOhm ke GND menjamin pin ini selalu LOW saat booting.

---

## 9. MASTER PINOUT MAP ESP32 (TABEL RANGKUMAN TOTAL)

Gunakan tabel ini untuk mengecek ulang kabel satu per satu sebelum menyalakan daya:

```
             +---------------------------------------------------------+
             |                ESP32 DEVKIT V1 (30 PIN)                 |
             +---------------------------------------------------------+
   KIRI (Left Pins)                                   KANAN (Right Pins)
 +-----------------------+-------+           +-------+-------------------------+
 | Fungsi Komponen       | Pin   |           | Pin   | Fungsi Komponen         |
 +-----------------------+-------+           +-------+-------------------------+
 | Rel 3.3V (Semua Sensor| 3V3   |           | VIN   | Rel 5.0V (Output MT3608)|
 | Rel GND (Ground Total)| GND   |           | GND   | Rel GND (Ground Total)  |
 | OneWire 3x DS18B20    | GPIO 4|           | GPIO13| MicroSD CS              |
 | TFT LCD CS            | GPIO 5|           | GPIO12| Buzzer 5V (+ 1k Pulldown|
 | *(Tidak dipakai)*     | GPIO 2|           | GPIO14| Touchscreen T_CS        |
 | *(Tidak dipakai)*     | GPIO 0|           | GPIO27| TFT LCD DC/RS           |
 | GPS NEO-6M RX2 (TX)   | GPIO16|           | GPIO26| TFT LCD RST             |
 | GPS NEO-6M TX2 (RX)   | GPIO17|           | GPIO25| LED Merah (+ 330 Ohm)   |
 | SPI SCK (Clock Total) │ GPIO18│           │ GPIO33│ LED Kuning (+ 330 Ohm)  │
 | SPI MISO (Data Masuk) │ GPIO19│           │ GPIO32│ LED Hijau (+ 330 Ohm)   │
 | I2C SDA (BME/MPU/RTC) │ GPIO21│           │ GPIO35│ MC-38 Pintu (+ 4.7k Pull|
 | I2C SCL (BME/MPU/RTC) │ GPIO22│           │ GPIO34│ Tombol 1: BACK          │
 | SPI MOSI (Data Keluar)│ GPIO23│           │ GPIO36│ Tombol 2: SELECT (VP)   │
 | *(Tidak dipakai)*     │ GPIO15│           │ GPIO39│ Tombol 3: CONFIRM (VN)  │
 +-----------------------+-------+           +-------+-------------------------+
```

---

## 10. URUTAN TES PENYALAAN PERTAMA KALI

1. **JANGAN COLOK BATERAI/ADAPTOR DULU.**
2. Gunakan Multimeter (Mode Continuity Bip):
   - Tempelkan tester hitam ke **GND**, tester merah ke **Rel 3.3V** -> **Pastikan TIDAK bunyi bip (tidak korslet)**.
   - Tempelkan tester hitam ke **GND**, tester merah ke **Rel 5.0V** -> **Pastikan TIDAK bunyi bip (tidak korslet)**.
3. Colokkan kabel USB dari laptop ke port USB ESP32 terlebih dahulu (uji firmware).
4. Jika lampu power ESP32 menyala merah terang dan layar TFT menyala tanpa panas berlebih, berarti wiring kamu **100% SUKSES DAN AMAN**!
