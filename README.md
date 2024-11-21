![Elektrobotika 2024](https://github.com/Faizyee/Elektrobotika_2024/blob/a62c9610bb55eec299da8ea1fce67014cdcda204/fp.png)

# Elektrobotika 2024
Workshop And Robotic Competition

Oleh:
- D3 Teknik Elektronika
- UKM Robotika

[Politeknik Harapan Bersama Tegal](https://www.poltekharber.ac.id)

## Materi

### Instalasi

1. Instal Arduino IDE
   
   [Unduh Arduino IDE](https://www.arduino.cc/en/software)

3. Instal board ESP8266 dan driver CH340

   Copy board ESP8266 :
   >  ```http://arduino.esp8266.com/stable/package_esp8266com_index.json```

   [Unduh driver CH340](https://sparks.gogo.co.nz/ch340.html)

5. Instal library WebSocket, JsonArduino, bersControlV1

   - WebSockets unduh di Arduino IDE atau [disini (WebSockets)](https://github.com/Links2004/arduinoWebSockets)
   
   - JsonArduino unduh di Arduino IDE atau [disini (JsonArduino)](https://github.com/bblanchon/ArduinoJson)
   
   - bersControlV1 unduh [disini (bersControlV1)](https://github.com/Faizyee/BersControl/archive/refs/heads/main.zip)

### Pemrograman

```ino
#include <bersControlV1.h>

// Pin untuk motor dan kontrol robot
const int EA = D5;  // (EnA) Pin untuk motor kiri depan (PWM)
const int EB = D6;  // (EnB) Pin untuk motor kanan depan (PWM)
const int RB = D0;  // (IN1) Pin untuk motor kanan belakang (kontrol digital)
const int RF = D1;  // (IN2) Pin untuk motor kanan depan (kontrol digital)
const int LB = D2;  // (IN3) Pin untuk motor kiri belakang (kontrol digital)
const int LF = D3;  // (IN4) Pin untuk motor kiri depan (kontrol digital)

bersControlV1 control;  // Objek bersControlV1 untuk komunikasi WebSocket

// Fungsi yang menangani event WebSocket dan data yang diterima dari client
void onEventBersControl(const BersSignal& data) {
    // Mengecek jika status kode data adalah 0 (berarti data valid)
    if (data.status.Code == 0) {
      // Mengambil data dalam format JSON dari pesan yang diterima
      JsonDocument jsonData = data.output.Json;

      // Mengambil status perintah untuk setiap motor dan kecepatan
      bool LF_p = jsonData["data"]["left"]["up"].as<bool>();     // Status motor kiri depan (gerak maju)
      bool LB_p = jsonData["data"]["left"]["down"].as<bool>();   // Status motor kiri belakang (gerak mundur)
      bool RF_p = jsonData["data"]["right"]["up"].as<bool>();    // Status motor kanan depan (gerak maju)
      bool RB_p = jsonData["data"]["right"]["down"].as<bool>();  // Status motor kanan belakang (gerak mundur)
      int speed = jsonData["data"]["speed"].as<int>();           // Kecepatan motor (PWM)

      // Mengatur kecepatan kedua motor (kiri dan kanan) berdasarkan data yang diterima
      analogWrite(EA, speed);  // Motor kiri depan
      analogWrite(EB, speed);  // Motor kanan depan

      // LF           RF
      // LB           RB
      //     |||||||    

      // Mengontrol motor kanan depan dan belakang
      if (RF_p && !RB_p) {
        digitalWrite(RF, HIGH);  // Motor kanan depan maju
        digitalWrite(RB, LOW);   // Motor kanan belakang berhenti
      } else if (!RF_p && RB_p) {
        digitalWrite(RF, LOW);   // Motor kanan depan berhenti
        digitalWrite(RB, HIGH);  // Motor kanan belakang mundur
      } else {
        digitalWrite(RF, LOW);  // Mematikan motor kiri depan
        digitalWrite(RB, LOW);  // Mematikan motor kiri belakang
      }

      // Mengontrol motor kiri depan dan belakang
      if (LF_p && !LB_p) {
        digitalWrite(LF, HIGH);  // Motor kiri depan maju
        digitalWrite(LB, LOW);   // Motor kiri belakang berhenti
      } else if (!LF_p && LB_p) {
        digitalWrite(LF, LOW);   // Motor kiri depan berhenti
        digitalWrite(LB, HIGH);  // Motor kiri belakang mundur
      } else {
        digitalWrite(LF, LOW);  // Mematikan motor kiri depan
        digitalWrite(LB, LOW);  // Mematikan motor kiri belakang
      }
    }
}

// Fungsi setup untuk inisialisasi hardware dan koneksi
void setup() {
  // Menetapkan mode pin untuk motor dan LED
  pinMode(EA, OUTPUT);           // Motor kiri depan
  pinMode(EB, OUTPUT);           // Motor kanan depan
  pinMode(RB, OUTPUT);           // Motor kanan belakang
  pinMode(RF, OUTPUT);           // Motor kanan depan
  pinMode(LB, OUTPUT);           // Motor kiri belakang
  pinMode(LF, OUTPUT);           // Motor kiri depan
  pinMode(LED_BUILTIN, OUTPUT);  // LED built-in untuk indikator status

  // Mengatur kecepatan awal kedua motor pada level 180 (range 0-255)
  analogWrite(EA, 180);  // Motor kiri depan
  analogWrite(EB, 180);  // Motor kanan depan

  // Inisialisasi koneksi WebSocket dan menghubungkan dengan SSID, Password, dan Mode Access Point (true) atau Station (false)
  control.begin("Player 1", "123456789", true);  // SSID: Player1, Password: 123456789, Mode: Access Point

  // Menetapkan fungsi event handler untuk menangani data yang diterima
  control.onEvent(onEventBersControl);
}

// Fungsi loop yang dijalankan terus menerus untuk menangani komunikasi
void loop() {
  control.loop();  // Menjaga komunikasi WebSocket tetap aktif
}
```

### Rangkaian

![Rangkaian](https://github.com/Faizyee/Elektrobotika_2024/blob/fe2750ef85d2cade316f6d106fdc46b584f7e28b/sp.png)

### Sistem Kontrol

![Buka kontrol sekarang](https://faizyee.github.io/Elektrobotika_2024)