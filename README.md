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

   - WebSockets unduh di Arduino IDE
   
   - JsonArduino unduh di Arduino IDE
   
   - bersControlV1 unduh [disini (bersControlV1)](https://github.com/Faizyee/BersControl/archive/refs/heads/main.zip)

### Pemrograman

#### ESP8266:
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
  pinMode(EA, OUTPUT);           // Motor kiri kecepatan
  pinMode(EB, OUTPUT);           // Motor kanan kecepatan
  pinMode(RB, OUTPUT);           // Motor kanan belakang
  pinMode(RF, OUTPUT);           // Motor kanan depan
  pinMode(LB, OUTPUT);           // Motor kiri belakang
  pinMode(LF, OUTPUT);           // Motor kiri depan

  // Mengatur kecepatan awal kedua motor pada level 180 (range 0-255)
  analogWrite(EA, 180);  // Motor kiri depan
  analogWrite(EB, 180);  // Motor kanan depan

  // Inisialisasi koneksi WebSocket dan menghubungkan dengan SSID, Password, dan Mode Access Point (true) atau Station (false)
  control.begin("Player 10", "123456789", true);  // SSID: Player1, Password: 123456789, Mode: Access Point
  control.setMaxConnect(1);

  // Menetapkan fungsi event handler untuk menangani data yang diterima
  control.onEvent(onEventBersControl);
}

// Fungsi loop yang dijalankan terus menerus untuk menangani komunikasi
void loop() {
  control.loop();  // Menjaga komunikasi WebSocket tetap aktif
}
```

#### Web Control
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Elektrobotika 2024</title>
    <style>
        /* Reset default browser styles untuk elemen dan mengatur agar elemen tidak memiliki shadow */
        * {
            outline: none;
            -webkit-appearance: none;
            appearance: none;
            -webkit-box-shadow: none;
            box-shadow: none;
            user-select: none;
        }

        /* Style untuk body agar tampil di tengah dengan background warna lembut */
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            background-color: #f4f4f9;
        }

        /* Style untuk kontainer awal (start-container) */
        .start-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: space-between;
            width: 100%;
            max-width: 600px;
            padding: 20px;
            box-sizing: border-box;
            gap: 10px;
        }

        /* Style untuk kontainer utama (main-container) setelah terkoneksi */
        .main-container {
            display: flex;
            flex-direction: row;
            align-items: center;
            justify-content: space-between;
            width: 100%;
            max-width: 600px;
            padding: 20px;
            box-sizing: border-box;
        }

        /* Style untuk kolom */
        .column {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }

        /* Style tombol default */
        .button {
            width: 100px;
            height: 100px;
            font-size: 16px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            text-align: center;
        }

        /* Warna tombol saat ditekan */
        .button.active {
            background-color: #0056b3;
        }

        /* Style untuk seekbar */
        .seekbar-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }

        /* Style khusus untuk seekbar vertikal */
        .seekbar {
            writing-mode: bt-lr;
            -webkit-appearance: slider-vertical;
            height: 180px;
            width: 20px;
        }

        /* Style untuk teks seekbar */
        #seekbar-value {
            font-size: 16px;
            font-weight: bold;
        }

        /* Class tersembunyi untuk elemen yang akan diaktifkan/dinonaktifkan */
        .hidden {
            display: none;
        }

        /* Style tombol "Mulai" */
        #btn-start {
            width: 100px;
            height: 40px;
            font-size: 16px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            text-align: center;
        }

        /* Style input teks untuk IP dan port */
        input[type="text"] {
            padding: 12px 20px;
            font-size: 16px;
            border: 2px solid #ddd;
            border-radius: 8px;
            box-sizing: border-box;
            text-align: center;
            min-width: 150px;
            max-width: 100%;
        }

        /* Style untuk placeholder input teks */
        input[type="text"]::placeholder {
            color: #aaa;
            font-style: italic;
            text-align: center;
        }

        /* Style untuk pesan error */
        .error-message {
            color: #000000;
            font-size: 14px;
            margin-top: 5px;
            text-align: center;
        }
    </style>
</head>

<body>
    <!-- Bagian awal untuk memasukkan IP dan port -->
    <div id="start" class="start-container">
        <span id="error-message" class="error-message"></span>
        <input type="text" id="textInput" placeholder="IP Address dan port">
        <button id="btn-start" type="button" onclick="connect()">Mulai</button>
    </div>

    <!-- Bagian utama untuk kontrol setelah tersambung -->
    <div id="main" class="main-container hidden">
        <div class="column">
            <button class="button" id="btn-left-up">Atas</button>
            <button class="button" id="btn-left-down">Bawah</button>
        </div>
        <div class="seekbar-container">
            <input type="range" min="0" max="255" value="180" id="seekbar" class="seekbar">
            <label id="seekbar-value">180</label>
        </div>
        <div class="column">
            <button class="button" id="btn-right-up">Atas</button>
            <button class="button" id="btn-right-down">Bawah</button>
        </div>
    </div>

    <script>
        let ws; // Objek WebSocket

        // Fungsi untuk memulai koneksi ke WebSocket server
        function connect() {
            var input = document.getElementById('textInput');
            var errorMessage = document.getElementById('error-message');

            // Validasi input apakah kosong
            if (input.value.trim() === '') {
                errorMessage.textContent = "IP tidak boleh kosong!";
                errorMessage.style.color = "#e74c3c";
            } else {
                // Inisialisasi WebSocket
                if (typeof ws == 'undefined') {
                    ws = new WebSocket("ws://" + input.value);
                    errorMessage.textContent = "Menghubungkan...";
                    errorMessage.style.color = "#000000";

                    // Callback saat WebSocket berhasil tersambung
                    ws.onopen = () => {
                        errorMessage.textContent = "Tersambung!";
                        errorMessage.style.color = "#4ae73c";
                        // Tampilkan bagian utama setelah koneksi
                        setTimeout(function () {
                            document.getElementById('main').classList.remove('hidden');
                            document.getElementById('start').classList.add('hidden');
                        }, 1000);
                    };

                    // Callback saat koneksi WebSocket terputus
                    ws.onclose = () => {
                        errorMessage.textContent = "Gagal terkoneksi!";
                        errorMessage.style.color = "#e74c3c";
                    };

                    // Callback saat terjadi error di WebSocket
                    ws.onerror = (error) => {
                        errorMessage.textContent = "Error: " + error.message;
                        errorMessage.style.color = "#e74c3c";
                    };
                }
            }
        }

        // Objek untuk menyimpan state tombol dan seekbar
        const buttonState = {
            left: { up: false, down: false },
            right: { up: false, down: false },
            speed: 180, // Nilai awal speed dari seekbar
        };

        // Fungsi untuk mengirim data state ke server
        function sendState() {
            const data = { data: buttonState };
            if (typeof ws != 'undefined' && ws.readyState === WebSocket.OPEN) {
                ws.send(JSON.stringify(data));
            }
        }

        // Event listener untuk seekbar
        const seekbar = document.getElementById("seekbar");
        const seekbarValue = document.getElementById("seekbar-value");
        seekbar.addEventListener("input", () => {
            seekbarValue.textContent = seekbar.value; // Perbarui tampilan nilai
            buttonState.speed = seekbar.value; // Perbarui state speed
            sendState(); // Kirim state baru ke server
        });

        // Fungsi untuk menangani tombol yang ditekan
        function handlePress(button, name, stateKey, direction) {
            button.classList.add("active");
            buttonState[stateKey][direction] = true;
            sendState(); // Kirim state baru
        }

        // Fungsi untuk menangani tombol yang dilepas
        function handleRelease(button, name, stateKey, direction) {
            button.classList.remove("active");
            buttonState[stateKey][direction] = false;
            sendState(); // Kirim state baru
        }

        // Tambahkan event listener ke tombol
        const btnLeftUp = document.getElementById("btn-left-up");
        const btnLeftDown = document.getElementById("btn-left-down");
        const btnRightUp = document.getElementById("btn-right-up");
        const btnRightDown = document.getElementById("btn-right-down");

        btnLeftUp.addEventListener("touchstart", () => handlePress(btnLeftUp, "Left Up", "left", "up"), { passive: true });
        btnLeftUp.addEventListener("touchend", () => handleRelease(btnLeftUp, "Left Up", "left", "up"), { passive: true });

        // Tambahkan event untuk tombol lainnya (mirip seperti btnLeftUp)
        btnLeftDown.addEventListener("touchstart", () => handlePress(btnLeftDown, "Left Down", "left", "down"), { passive:true });
btnLeftDown.addEventListener("touchend", () => handleRelease(btnLeftDown, "Left Down", "left", "down"), { passive: true });

btnRightUp.addEventListener("touchstart", () => handlePress(btnRightUp, "Right Up", "right", "up"), { passive: true });
btnRightUp.addEventListener("touchend", () => handleRelease(btnRightUp, "Right Up", "right", "up"), { passive: true });

btnRightDown.addEventListener("touchstart", () => handlePress(btnRightDown, "Right Down", "right", "down"), { passive: true });
btnRightDown.addEventListener("touchend", () => handleRelease(btnRightDown, "Right Down", "right", "down"), { passive: true });
    </script>
</body>

</html>
```

### Rangkaian

![Rangkaian](https://github.com/Faizyee/Elektrobotika_2024/blob/fe2750ef85d2cade316f6d106fdc46b584f7e28b/sp.png)

### Sistem Kontrol

Data yang akan terkirim adalah:
```js
{
  data: {
    left: {
      up: true/false,
      down: true/false
    },
    right: {
      up: true/false,
      down: true/false
    },
    speed: 0-255
  }
}
```

[Buka kontrol sekarang](https://faizyee.github.io/Elektrobotika_2024)