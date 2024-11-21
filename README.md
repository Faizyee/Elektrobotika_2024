![Elektrobotika 2024](https://github.com/Faizyee/Elektrobotika_2024/blob/a62c9610bb55eec299da8ea1fce67014cdcda204/fp.png)

# Elektrobotika 2024
Workshop And Robotic Competition

Oleh:
- Elektronika
- Robotika

Politeknik Harapan Bersama Tegal

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

bersControlV1 control;

void eventControl(const DataBers& data) {

}

void setup() {
  control.begin("SSID", "PASSWORD");
  control.setMaxClient(1);
  control.onEvent(eventControl);
}

void loop() {
  control.loop();
}
```

Hello