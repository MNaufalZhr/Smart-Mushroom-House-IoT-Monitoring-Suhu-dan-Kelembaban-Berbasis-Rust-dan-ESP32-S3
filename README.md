# Smart-Mushroom-House-IoT-Monitoring-Suhu-dan-Kelembaban-Berbasis-Rust-dan-ESP32-S3
Perkembangan teknologi digital telah membawa perubahan besar dalam berbagai bidang, termasuk pertanian. Salah satu inovasi penting adalah Internet of Things (IoT), yaitu konsep yang menghubungkan perangkat seperti sensor dan mikrokontroler melalui internet untuk memantau dan mengendalikan kondisi secara otomatis dan real-time.
Dalam budidaya jamur, suhu dan kelembaban merupakan faktor utama yang menentukan pertumbuhan optimal. Oleh karena itu, diperlukan sistem pemantauan otomatis yang mampu menjaga kestabilan lingkungan kumbung jamur.

Proyek Mushroom House dirancang untuk memenuhi kebutuhan tersebut dengan menggunakan mikrokontroler ESP32-S3, sensor DHT22, dan bahasa pemrograman Rust yang unggul dalam keamanan memori dan efisiensi tinggi. Sistem ini juga dilengkapi fitur Over-The-Air (OTA) untuk pembaruan firmware jarak jauh melalui internet, serta menggunakan protokol MQTT dan platform ThingsBoard sebagai media komunikasi dan visualisasi data. Kombinasi teknologi tersebut menghasilkan solusi IoT yang efisien, andal, dan mudah dikembangkan untuk pemantauan lingkungan kumbung jamur.
# NAMA KELOMPOK 16 :
1. Muhammad Naufal Zuhair (2042231005)
2. Ahmad Radhy, S.Si., M.Si (Supervisor)
# Pengertian Setiap Tools

---

##  Internet of Things (IoT)


**Internet of Things (IoT)** adalah konsep jaringan yang menghubungkan berbagai perangkat agar dapat saling berkomunikasi melalui internet untuk bertukar data antara dunia fisik dan digital.  
IoT terdiri dari tiga komponen utama:
- **Jaringan Komunikasi** – mengirimkan data ke server  
- **Platform Aplikasi Thingsboards** – menampilkan hasil pengolahan data  

Protokol umum yang digunakan adalah **MQTT** karena efisien dan mendukung komunikasi *asynchronous*.  
Salah satu contoh penerapan IoT adalah **Mushroom House**, sistem rumah pintar yang mengintegrasikan sensor dan cloud untuk memantau serta mengendalikan perangkat secara efisien.

---

## Mikrokontroler ESP32-S3


**ESP32-S3** merupakan mikrokontroler *System-on-Chip (SoC)* yang dirancang untuk sistem IoT.  
Spesifikasi utama:
- CPU: *Xtensa LX7 Dual-Core* hingga 240 MHz  
- Dukungan AI dengan *vector instruction*  
- Fitur keamanan: *Secure Boot* & *Flash Encryption*  
- Antarmuka: UART, SPI, I2C, I2S, PWM  

ESP32-S3 digunakan pada proyek **Mushroom House** untuk membaca data sensor DHT22 dan mengirimnya ke platform **ThingsBoard Cloud** via protokol **MQTT**.  
Bahasa pemrograman **Rust** digunakan melalui pustaka `esp-idf-svc` dan `embedded-svc`.

---

## Bahasa Pemrograman Rust untuk Embedded System


**Rust** adalah bahasa pemrograman modern yang dirancang untuk efisiensi tinggi dan keamanan memori.  
Fitur utamanya:
- Konsep *ownership* dan *borrowing* untuk mencegah *data race*  
- Efisiensi setara C, namun lebih aman  
- Mendukung *asynchronous programming*  

Dalam **Mushroom House**, Rust digunakan untuk:
- Koneksi Wi-Fi  
- Komunikasi MQTT  
- Pembacaan sensor DHT22  
- Pembaruan firmware *Over-The-Air (OTA)*  

---

## Sensor DHT22


**DHT22 (AM2302)** adalah sensor digital untuk mengukur suhu dan kelembaban.  
Karakteristik:
- Rentang suhu: −40°C hingga 80°C  
- Akurasi tinggi & stabilitas jangka panjang  
- Interval pembaruan: 2 detik  

Sensor ini mengirim data ke **ESP32-S3** melalui komunikasi *single-wire digital* dan hasilnya ditampilkan di **ThingsBoard Cloud** menggunakan protokol **MQTT**.

---

## Protokol MQTT


**MQTT (Message Queuing Telemetry Transport)** adalah protokol komunikasi ringan berbasis arsitektur *publish/subscribe*.  
Kelebihan:
- Penggunaan bandwidth efisien  
- Komunikasi tetap stabil meski jaringan tidak ideal  
- Mendukung QoS level 0–2  

Dalam **Mushroom House**, MQTT digunakan untuk:
- Mengirim data suhu dan kelembaban ke **ThingsBoard Cloud**  
- Mengatur pembaruan firmware dengan QoS tinggi  

---

## Platform ThingsBoard Cloud


**ThingsBoard** adalah platform *open-source* IoT untuk mengelola perangkat, menyimpan data sensor, dan menampilkan informasi lewat *dashboard interaktif*.  
Fitur unggulan:
- Mendukung protokol MQTT, HTTP, dan CoAP  
- *Time-series database*  
- *Rule engine* untuk otomatisasi  
- Dukungan TLS/SSL untuk keamanan data  

Dalam proyek **Mushroom House**, ThingsBoard digunakan untuk memantau suhu dan kelembaban secara *real-time* serta melakukan pembaruan firmware OTA.

---

## Over-The-Air (OTA)


**OTA (Over-The-Air)** memungkinkan pembaruan firmware IoT secara jarak jauh tanpa perlu *flashing* manual.  
Arsitektur utama:
- Server firmware  
- Broker komunikasi  
- Klien IoT  

Keunggulan:
- Aman dengan enkripsi TLS  
- Mendukung *rollback* jika pembaruan gagal  
- Efisien untuk banyak perangkat  

Pada **Mushroom House**, OTA dijalankan menggunakan Rust melalui perintah **RPC** dari ThingsBoard.

---

## Firmware pada Sistem IoT


**Firmware** adalah perangkat lunak tertanam yang mengatur operasi dasar perangkat IoT seperti sensor dan komunikasi jaringan.  
Dalam proyek **Mushroom House**, firmware Rust mengatur:
- Inisialisasi Wi-Fi  
- Koneksi MQTT  
- Pengambilan data sensor DHT22  
- Pembaruan otomatis (OTA)  

Rust dipilih karena efisien, aman dari *data race*, dan mendukung *asynchronous non-blocking operation* untuk performa tinggi.

---
# STEP BY STEP PERANCANGAN SISTEM
## 1. Konfigurasi Sensor
   ### Main.rs Pada Sensor DHT22
   ```Rust
use std::{thread, time::Duration, sync::{Arc, atomic::{AtomicBool, Ordering}}};
use anyhow::{Result, Error};
use chrono::{Duration as ChronoDuration, NaiveDateTime, Utc, TimeZone}; // WARNING FIX: Removed unused DateTime import
use dht_sensor::dht22::Reading;
use dht_sensor::DhtReading;
use esp_idf_svc::{
    eventloop::EspSystemEventLoop,
    hal::{
        delay::Ets,
        // WARNING FIX: Removed unused Input, Output, Pin
        gpio::{PinDriver}, 
        prelude::*,
    },
    log::EspLogger,
    mqtt::client::*,
    nvs::EspDefaultNvsPartition,
    sntp,
    systime::EspSystemTime,
    wifi::*,
    ota::EspOta,
    http::client::EspHttpConnection,
};
use embedded_svc::{
    mqtt::client::QoS,
    // WARNING FIX: Removed unused Request
    http::client::Client, 
    io::Read, // This is embedded_svc::io::Read, required for ota_process
};
use heapless::String;
use serde_json::json;
// use rand::Rng; // ⚠️ CLEANUP: Dihapus karena tidak digunakan

// --- Konfigurasi Firmware & Device ---
const CURRENT_FIRMWARE_VERSION: &str = "PaceP-s3-v2.0";
// 📌 Menggunakan ThingsBoard Cloud
const TB_MQTT_URL: &str = "mqtt://mqtt.thingsboard.cloud:1883";
const THINGSBOARD_TOKEN: &str = "czxYDbyMCDnFqtM8CPGM"; // Token Device ThingsBoard

// 3. EXTERN C FUNCTION
// ✅ FIX E0425: Menambahkan kembali deklarasi fungsi C
extern "C" {
    fn esp_restart();
}

// =========================================================================================
// --- MQTT Client State (Global Access) ---
static mut MQTT_CLIENT: Option<EspMqttClient<'static>> = None;

fn get_mqtt_client() -> Option<&'static mut EspMqttClient<'static>> {
    unsafe {
        MQTT_CLIENT.as_mut().map(|c| {
            // SAFETY: Transmute untuk mengatasi batasan lifetime 'static dari EspMqttClient
            std::mem::transmute::<&mut EspMqttClient<'_>, &mut EspMqttClient<'static>>(c)
        })
    }
}

// Fungsi untuk mengirim telemetry fw_state
fn publish_fw_state(state: &str) {
    let payload = format!("{{\"fw_state\":\"{}\"}}", state);
    log::info!("➡️ Mengirim telemetry fw_state: {}", payload);

    if let Some(client) = get_mqtt_client() {
        // ✅ FIX 1: Mengubah QoS dari ExactlyOnce (QoS 2) menjadi AtLeastOnce (QoS 1)
        if let Err(e) = client.publish(
            "v1/devices/me/telemetry",
            QoS::AtLeastOnce, 
            false,
            payload.as_bytes(),
        ) {
            log::error!("⚠️ Gagal kirim fw_state {}: {:?}", state, e);
        }
    } else {
        log::error!("⚠️ MQTT client belum siap untuk kirim fw_state {}", state);
    }
}

// Mengirim versi firmware saat ini (CRUCIAL UNTUK OTA)
fn publish_fw_version() {
    let payload = format!("{{\"fw_version\":\"{}\"}}", CURRENT_FIRMWARE_VERSION);
    log::info!("➡️ Mengirim Current FW Version: {}", payload);

    if let Some(client) = get_mqtt_client() {
        if let Err(e) = client.publish(
            "v1/devices/me/telemetry",
            QoS::AtLeastOnce,
            false,
            payload.as_bytes(),
        ) {
            log::error!("⚠️ Gagal kirim fw_version: {:?}", e);
        }
    } else {
        log::error!("⚠️ MQTT client belum siap untuk kirim fw_version");
    }
}

// Fungsi untuk mengirim RPC response ke ThingsBoard
fn send_rpc_response(request_id: &str, status: &str) {
    let topic = format!("v1/devices/me/rpc/response/{}", request_id);
    log::info!("➡️ Mengirim RPC response ke: {}", topic);

    let payload = format!("{{\"status\":\"{}\"}}", status);

    if let Some(client) = get_mqtt_client() {
        if let Err(e) = client.publish(
            topic.as_str(),
            QoS::AtLeastOnce,
            false,
            payload.as_bytes(),
        ) {
            log::error!("⚠️ Gagal kirim RPC response: {:?}", e);
        }
    } else {
        log::error!("⚠️ MQTT client belum siap untuk kirim RPC response");
    }
}

// 5. OTA PROCESS FUNCTION
fn ota_process(url: &str) {
    log::info!("📥 Mulai OTA dari URL: {}", url);
    publish_fw_state("DOWNLOADING");

    // Jeda 500ms untuk memastikan pesan "DOWNLOADING" terkirim oleh event loop MQTT
    // sebelum proses HTTP download yang berat dimulai dan memblokir network.
    thread::sleep(Duration::from_millis(500));

    match EspOta::new() {
        Ok(mut ota) => {
            let http_config = esp_idf_svc::http::client::Configuration {
                ..Default::default()
            };

            let conn = match EspHttpConnection::new(&http_config) {
                Ok(c) => c,
                Err(e) => {
                    log::error!("⚠️ Gagal buat koneksi HTTP: {:?}", e);
                    publish_fw_state("FAILED");
                    return;
                }
            };

            let mut client = Client::wrap(conn);
            let request = match client.get(url) {
                Ok(r) => r,
                Err(e) => {
                    log::error!("⚠️ Gagal buat HTTP GET: {:?}", e);
                    publish_fw_state("FAILED");
                    return;
                }
            };

            let mut response = match request.submit() {
                Ok(r) => r,
                Err(e) => {
                    log::error!("⚠️ Gagal submit request: {:?}", e);
                    publish_fw_state("FAILED");
                    return;
                }
            };

            if response.status() < 200 || response.status() >= 300 {
                log::error!("⚠️ HTTP request gagal. Status code: {}", response.status());
                publish_fw_state("FAILED");
                return;
            }

            let mut buf = [0u8; 1024];
            let mut update = match ota.initiate_update() {
                Ok(u) => u,
                Err(e) => {
                    log::error!("⚠️ Gagal init OTA: {:?}", e);
                    publish_fw_state("FAILED");
                    return;
                }
            };

            loop {
                match response.read(&mut buf) {
                    Ok(0) => break,
                    Ok(size) => {
                        if let Err(e) = update.write(&buf[..size]) {
                            log::error!("⚠️ Gagal tulis OTA: {:?}", e);
                            publish_fw_state("FAILED");
                            return;
                        }
                    }
                    Err(e) => {
                        log::error!("⚠️ HTTP read error: {:?}", e);
                        publish_fw_state("FAILED");
                        return;
                    }
                }
            }

            publish_fw_state("VERIFYING");

            if let Err(e) = update.complete() {
                log::error!("⚠️ OTA complete error: {:?}", e);
                publish_fw_state("FAILED");
                return;
            }

            log::info!("✅ OTA selesai, restart...");
            publish_fw_state("SUCCESS");

            // Jeda 1 detik agar pesan SUCCESS terkirim
            thread::sleep(Duration::from_secs(1));

            unsafe { esp_restart(); }
        }
        Err(e) => {
            log::error!("⚠️ Gagal init OTA: {:?}", e);
            publish_fw_state("FAILED");
        }
    }
}

// 6. MAIN FUNCTION
fn main() -> Result<(), Error> {
    // --- Inisialisasi dasar ---
    esp_idf_svc::sys::link_patches();
    EspLogger::initialize_default();
    log::info!("🚀 Program dimulai, Versi FW: {} - 🔥 FIRMWARE AKTIF!", CURRENT_FIRMWARE_VERSION);

    // --- Inisialisasi perangkat ---
    let peripherals = Peripherals::take().unwrap();
    let sysloop = EspSystemEventLoop::take()?;
    let nvs = EspDefaultNvsPartition::take().unwrap();

    // --- Konfigurasi WiFi ---
    let mut wifi = EspWifi::new(peripherals.modem, sysloop.clone(), Some(nvs.clone()))?;

    let mut ssid: String<32> = String::new();
    ssid.push_str("No Internet").unwrap();

    let mut pass: String<64> = String::new();
    pass.push_str("tertolong123").unwrap();

    let wifi_config = Configuration::Client(ClientConfiguration {
        ssid,
        password: pass,
        auth_method: AuthMethod::WPA2Personal,
        ..Default::default()
    });

    wifi.set_configuration(&wifi_config)?;
    wifi.start()?;
    wifi.connect()?;

    // --- Tunggu sampai WiFi benar-benar aktif ---
    while !wifi.is_connected().unwrap() {
        log::info!("⏳ Menunggu koneksi WiFi...");
        thread::sleep(Duration::from_secs(1));
    }
    log::info!("✅ WiFi terhubung!");

    // Leak services to prevent drop/deinitialization
    let _wifi = Box::leak(Box::new(wifi));
    let _sysloop = Box::leak(Box::new(sysloop));
    let _nvs = Box::leak(Box::new(nvs));

    // --- Sinkronisasi waktu via NTP ---
    log::info!("🌐 Sinkronisasi waktu NTP...");
    let sntp = sntp::EspSntp::new_default()?;

    loop {
        let status = sntp.get_sync_status();
        if status == sntp::SyncStatus::Completed {
            log::info!("✅ Waktu berhasil disinkronkan dari NTP");
            break;
        } else {
            log::info!("⏳ Menunggu sinkronisasi NTP...");
            thread::sleep(Duration::from_secs(1));
        }
    }

    // Delay tambahan agar waktu stabil
    thread::sleep(Duration::from_secs(5));

    // --- Konfigurasi MQTT (ThingsBoard Cloud) ---
    let mqtt_config = MqttClientConfiguration {
        client_id: Some("esp32-rust-ota"),
        username: Some(THINGSBOARD_TOKEN), // Menggunakan const THINGSBOARD_TOKEN
        password: None,
        keep_alive_interval: Some(Duration::from_secs(30)),
        ..Default::default()
    };

    let mqtt_connected = Arc::new(AtomicBool::new(false));

    // --- MQTT Callback Handler (untuk menangani RPC OTA) ---
    let mqtt_callback = {
        let mqtt_connected = mqtt_connected.clone();

        move |event: EspMqttEvent<'_>| {
            use esp_idf_svc::mqtt::client::EventPayload;

            match event.payload() {
                EventPayload::Connected(_) => {
                    log::info!("📡 MQTT connected");
                    mqtt_connected.store(true, Ordering::SeqCst);
                }
                EventPayload::Received { topic, data, .. } => {
                    let payload_str = std::str::from_utf8(data).unwrap_or("");
                    log::info!("📩 Payload diterima. Topic: {:?}, Data: {}", topic, payload_str);

                    if let Some(topic_str) = topic {
                        // ⚡ Logic RPC Request untuk OTA
                        if topic_str.starts_with("v1/devices/me/rpc/request/") {
                            let parts: Vec<&str> = topic_str.split('/').collect();
                            if let Some(request_id) = parts.last() {
                                log::info!("✅ Menerima RPC request_id: {}", request_id);

                                if let Ok(json) = serde_json::from_str::<serde_json::Value>(payload_str) {
                                    let mut ota_url_owned = None;

                                    // Cek apakah ada parameter ota_url di dalam RPC
                                    if let Some(url_str) = json.get("params").and_then(|p| p.get("ota_url")).and_then(|u| u.as_str()) {
                                        // ✅ FIX E0597: Kloning string agar thread baru memiliki data (owned)
                                        ota_url_owned = Some(url_str.to_owned());
                                    }

                                    if let Some(url) = ota_url_owned {
                                        log::info!("⚡ Dapat OTA URL dari RPC: {}", url);
                                        send_rpc_response(request_id, "success");
                                        
                                        // ✅ FIX 2: Mencegah Stack Overflow dengan mengalokasikan stack 10KB
                                        std::thread::Builder::new()
                                            .stack_size(10 * 1024) 
                                            .spawn(move || {
                                                // Pass reference (&str) to the owned String
                                                ota_process(&url);
                                            })
                                            .expect("Gagal membuat thread OTA");
                                            
                                        return;
                                    } else {
                                        log::warn!("⚠️ Payload RPC diterima, tetapi \"ota_url\" tidak ditemukan.");
                                    }
                                } else {
                                    log::error!("⚠️ Gagal mem-parse JSON payload RPC.");
                                }

                                send_rpc_response(request_id, "failure");
                            }
                        }
                    }
                }
                EventPayload::Disconnected => {
                    log::warn!("⚠️ MQTT Disconnected!");
                    mqtt_connected.store(false, Ordering::SeqCst);
                }
                _ => {}
            }
        }
    };

    // --- Inisialisasi MQTT Client (Non-static Callback) ---
    let client = loop {
        let res = unsafe {
            EspMqttClient::new_nonstatic_cb(
                TB_MQTT_URL,
                &mqtt_config,
                mqtt_callback.clone(),
            )
        };

        match res {
            Ok(c) => {
                unsafe { MQTT_CLIENT = Some(c) };

                if let Some(c_ref) = get_mqtt_client() {
                    while !mqtt_connected.load(Ordering::SeqCst) {
                        log::info!("⏳ Menunggu MQTT connect...");
                        thread::sleep(Duration::from_millis(500));
                    }
                    log::info!("📡 MQTT Connected!");

                    // Subscribe ke topik RPC untuk menerima perintah OTA
                    c_ref.subscribe("v1/devices/me/rpc/request/+", QoS::AtLeastOnce).unwrap();

                    // LAPORAN INI KRUSIAL UNTUK OTA (melaporkan versi dan status awal)
                    publish_fw_version();
                    publish_fw_state("IDLE");

                    break c_ref;
                } else {
                    log::error!("⚠️ Gagal mendapatkan referensi client setelah koneksi.");
                    thread::sleep(Duration::from_secs(5));
                    continue;
                }
            }
            Err(e) => {
                log::error!("⚠️ MQTT connect gagal: {:?}", e);
                thread::sleep(Duration::from_secs(5));
            }
        }
    };

    // --- Inisialisasi sensor DHT22 ---
    let mut pin = PinDriver::input_output_od(peripherals.pins.gpio4)?;
    let mut delay = Ets;

    // --- Loop utama kirim data (Interval 60 detik) ---
    loop {
        // Ambil waktu sekarang
        let systime = EspSystemTime {}.now();
        let secs = systime.as_secs() as i64;
        let nanos = systime.subsec_nanos();
        
        let naive = NaiveDateTime::from_timestamp_opt(secs, nanos as u32).unwrap_or(NaiveDateTime::from_timestamp_opt(0, 0).unwrap());
        // ✅ FIX E0308: Meminjam (&) nilai naive karena from_utc_datetime membutuhkan referensi.
        let utc_time = Utc.from_utc_datetime(&naive);
        let wib_time = utc_time + ChronoDuration::hours(7);
        // ✅ Fix Chrono API warning
        let ts_millis = naive.and_utc().timestamp_millis();
        let send_time_str = wib_time.format("%Y-%m-%d %H:%M:%S").to_string();

        // Baca sensor DHT22
        match Reading::read(&mut delay, &mut pin) {
            Ok(Reading {
                temperature,
                relative_humidity,
            }) => {
                // Siapkan payload JSON
                let payload = json!({
                    "send_time": send_time_str, // waktu kirim dari ESP (WIB)
                    "ts": ts_millis,            // waktu epoch (untuk analisis ThingsBoard)
                    "temperature": temperature,
                    "humidity": relative_humidity
                });

                let payload_str = payload.to_string();

                // Kirim data Telemetry
                match client.publish(
                    "v1/devices/me/telemetry",
                    QoS::AtLeastOnce, // Menggunakan QoS 1 untuk telemetry
                    false,
                    payload_str.as_bytes(),
                ) {
                    Ok(_) => log::info!("📤 Data terkirim (T: {}°C, H: {}%): {}", temperature, relative_humidity, payload_str),
                    Err(e) => log::error!("❌ Gagal publish ke MQTT: {:?}", e),
                }
            }
            Err(e) => log::error!("⚠️ Gagal baca DHT22: {:?}", e),
        }

        // Delay diubah menjadi 60 detik
        thread::sleep(Duration::from_secs(60));
    }
}
```
# ⚙️ Fungsi Program (`main.rs`)

## 🎯 Tujuan Utama
Firmware **IoT berbasis ESP32-S3** dengan fungsi utama untuk:

- 📡 Membaca suhu & kelembaban menggunakan **sensor DHT22**  
- ☁️ Mengirim data ke **ThingsBoard Cloud** melalui **MQTT**  
- 🔄 Melakukan **pembaruan firmware OTA (Over-The-Air)**  
- 🌐 Menjaga koneksi **WiFi** dan sinkronisasi waktu sistem (NTP)  

---

## 🔧 Fungsi Utama Program

### 1. 🧩 Inisialisasi Sistem
- Menyiapkan **logger**, event loop, dan NVS  
- Menghubungkan ESP32 ke **WiFi**  
- Menunggu hingga koneksi benar-benar stabil  

### 2. ⏱️ Sinkronisasi Waktu (NTP)
- Mengambil waktu real dari internet  
- Menetapkan waktu lokal (**WIB**) untuk data telemetry  

### 3. ☁️ Koneksi ke ThingsBoard (MQTT)
- Menghubungkan perangkat ke broker **ThingsBoard Cloud**  
- Mengirim versi dan status firmware saat startup  
- Berlangganan topik RPC untuk menerima perintah **OTA update**  

### 4. 🔁 Fitur OTA (Over-The-Air Update)
- Menerima perintah RPC berisi **URL firmware baru**  
- Mengunduh dan memverifikasi file firmware  
- Menulis ke partisi OTA lalu **restart otomatis**  
- Mengirim status pembaruan:  
  `DOWNLOADING → VERIFYING → SUCCESS/FAILED`  

### 5. 🌡️ Pembacaan Sensor DHT22
- Mengukur suhu dan kelembaban melalui **GPIO4**  
- Mengirim data telemetry dalam format **JSON** setiap 60 detik  

### 6. 🧠 Manajemen Status Firmware
- Melaporkan `fw_version` dan `fw_state` ke **ThingsBoard Cloud**  
- Memudahkan pemantauan versi dan status OTA perangkat  

---
### cargo.toml
```
Cargo.toml
[package]
name = "streamdht"
version = "0.1.0"
authors = ["zuhair"]
edition = "2021"
resolver = "2"
rust-version = "1.77"

[[bin]]
name = "streamdht"
harness = false # Do not use the built-in cargo test harness -> resolves rust-analyzer errors

[profile.release]
opt-level = "s"

[profile.dev]
debug = true       # Symbols are nice and they don't increase the size on Flash
opt-level = "z"

[features]
default = []
experimental = ["esp-idf-svc/experimental"]

[dependencies]
log             = "0.4"
esp-idf-svc     = "0.51"
rand            = "0.8"
anyhow          = "1.0"
heapless        = "0.8"
serde_json      = "1.0"
dht-sensor      = "0.2"
chrono          = { version = "0.4", features = ["clock"] }
embedded-svc    = { version = "0.28.1" } # FIX: Disinkronkan dengan esp-idf-svc 0.51

# --- Optional Embassy Integration ---
# esp-idf-svc = { version = "0.51", features = ["critical-section", "embassy-time-driver", "embassy-sync"] }

# If you enable embassy-time-driver, you MUST also add one of:
# embassy-time = { version = "0.4.0", features = ["generic-queue-8"] }
# embassy-executor = { version = "0.7", features = ["executor-thread", "arch-std"] }

# --- Temporary workaround for embassy-executor < 0.8 ---
# esp-idf-svc     = { version = "0.51", features = ["embassy-time-driver", "embassy-sync"] }
# critical-section = { version = "1.1", features = ["std"], default-features = false }

[build-dependencies]
embuild = "0.33"
```
# ⚙️ Fungsi & Tujuan Program (`Cargo.toml`)

## 🎯 Tujuan Utama
File `Cargo.toml` berfungsi sebagai **konfigurasi utama proyek Rust** pada firmware IoT ini.  
Tujuannya untuk:

- 🧱 Menentukan **informasi dasar proyek** seperti nama, versi, dan penulis  
- ⚙️ Mengatur **profil kompilasi** (debug dan release)  
- 🔗 Mendefinisikan **dependensi (library)** yang dibutuhkan  
- 🚀 Mengaktifkan fitur-fitur tambahan seperti **OTA, MQTT, dan sensor DHT22**  
- 🧩 Mengatur kompatibilitas dengan **Rust 1.77** dan sistem **ESP-IDF**  

---

## 🔧 Fungsi Utama Setiap Bagian

### 🏷️ `[package]`
Berisi metadata proyek:
- `name` → Nama proyek: **streamdht**
- `version` → Versi rilis firmware
- `authors` → Identitas pengembang
- `edition` dan `rust-version` → Menentukan standar bahasa Rust (Rust 2021)
- `resolver = 2` → Mengaktifkan dependency resolver modern untuk kompatibilitas lebih baik

---

### ⚙️ `[[bin]]`
- Menentukan nama file **binary output** (`streamdht`)
- `harness = false` → Menonaktifkan test harness bawaan Cargo agar tidak bentrok dengan sistem ESP-IDF

---

### 🚀 `[profile.release]` & `[profile.dev]`
Mengatur **optimasi dan debug level**:
- `release`: optimasi tinggi untuk efisiensi di perangkat ESP32-S3  
- `dev`: mode pengembangan dengan simbol debug aktif untuk mempermudah analisis log

---

### 🧩 `[features]`
- Menentukan fitur opsional yang bisa diaktifkan saat build
- Fitur `experimental` mengaktifkan opsi eksperimental dari `esp-idf-svc` (misalnya OTA, MQTT)

---

### 🧠 `[dependencies]`
Daftar pustaka utama yang digunakan:
- **log** → Sistem logging untuk debugging
- **esp-idf-svc** → SDK utama untuk ESP32 (WiFi, MQTT, OTA, NVS, SNTP)
- **anyhow** → Penanganan error yang fleksibel
- **heapless** → Struktur data ringan tanpa alokasi heap
- **serde_json** → Untuk encoding/decoding JSON (telemetry & RPC)
- **dht-sensor** → Membaca data suhu & kelembaban dari DHT22
- **chrono** → Manajemen waktu (sinkronisasi NTP dan timestamp)
- **embedded-svc** → Layanan tambahan untuk IoT (HTTP, MQTT, IO)
- **rand** → (opsional) untuk fungsi acak pada sistem tertanam

---

### 🛠️ `[build-dependencies]`
- **embuild** → Digunakan untuk menghubungkan sistem build Rust dengan **ESP-IDF framework**

---

## 💡 Kesimpulan
File `Cargo.toml` mengatur seluruh fondasi proyek **Rust IoT ESP32-S3**, mencakup:
- Struktur proyek  
- Ketergantungan pustaka  
- Mode kompilasi  
- Aktivasi fitur penting seperti MQTT, OTA, dan DHT22  

Dengan konfigurasi ini, firmware dapat dibangun secara efisien, stabil, dan kompatibel dengan ekosistem **ThingsBoard Cloud** serta perangkat keras **ESP32-S3**.
# TAMPILAN STREAMING DAN HASIL DATA
## STREAMING DATA DI TERMINAL DAN THINGSBOARDS
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/647847b1-b4d7-44b7-a62b-2182cc6f76a8" />

## BUKTI OTA BERHASIL DI UPDATE
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/cedbe3b0-12b9-4747-86c4-c9f9a5b97308" />
<img width="1208" height="267" alt="image" src="https://github.com/user-attachments/assets/89af9fce-7cdf-4ecc-aef2-5804aa92f049" />

## PERINTAH UNTUK MEMBUAT GRAFIK TEMPERATURE DAN HUMIDITY GNUPLOT PADA TERMINAL UBUNTU LINUX
<img width="820" height="621" alt="image" src="https://github.com/user-attachments/assets/0d8f715c-c652-454a-844c-d389a06289f6" />

### TAMPILAN GRAFIK TEMPERATURE DAN HUMIDITY PADA GNUPLOT
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/eef392ad-3479-4e6c-8d82-80c5ed9f2a80" />

## PERINTAH UNTUK MEMBUAT GRAFIK PERBANDINGAN LATENSI GNUPLOT PADA TERMINAL UBUNTU LINUX
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d49589bc-cdd5-442a-91ac-a7e289a66b1a" />

### TAMPILAN GRAFIK LATENSI THINGSBOARD VS ESP32-S3 PADA GNUPLOT 
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/db62c9c0-29f7-4da9-9c8c-200d3fd0b3fd" />

