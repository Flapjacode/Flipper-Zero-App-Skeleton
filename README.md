# Flipper-Zero-App-Skeleton

# 🐬 Flipper Zero App Skeleton 🐬 

## (UART GPIO) 
### Folder structure

Inside your Flipper firmware repo:

applications_user/esp_uart_tool/
 ├── esp_uart_tool.c
 └── application.fam
application.fam

This tells the firmware about your app.

Writing

App(
appid="esp_uart_tool",
name="ESP UART Tool",
apptype=FlipperAppType.EXTERNAL,
entry_point="esp_uart_tool_app",
stack_size=2048,
icon="A_Console_14",
)

esp_uart_tool.c

This example:

Opens UART

Sends "SCAN\n"

Prints responses on the screen

Writing

#include <furi.h>
#include <furi_hal.h>
#include <gui/gui.h>
#include <input/input.h>
#include <stdlib.h>

#define UART_CH FuriHalSerialIdUsart

static void draw_callback(Canvas* canvas, void* ctx) {
canvas_clear(canvas);
canvas_draw_str(canvas, 5, 15, "ESP UART Tool");
canvas_draw_str(canvas, 5, 35, "Press OK to scan");
}

int32_t esp_uart_tool_app(void* p) {

UNUSED(p);

Gui* gui = furi_record_open(RECORD_GUI);
ViewPort* view_port = view_port_alloc();

view_port_draw_callback_set(view_port, draw_callback, NULL);
gui_add_view_port(gui, view_port, GuiLayerFullscreen);

FuriHalSerialHandle* serial =
    furi_hal_serial_control_acquire(UART_CH);

furi_hal_serial_init(serial, 115200);

while(true) {

    InputEvent event;

    if(furi_message_queue_get(furi_hal_input_get_queue(),
                              &event,
                              FuriWaitForever) == FuriStatusOk) {

        if(event.type == InputTypePress &&
           event.key == InputKeyOk) {

            const char* cmd = "SCAN\n";

            furi_hal_serial_tx(serial,
                               (uint8_t*)cmd,
                               strlen(cmd));

        }
    }
}

gui_remove_view_port(gui, view_port);
view_port_free(view_port);
furi_record_close(RECORD_GUI);

return 0;

}

🔌 Flipper GPIO Wiring

Flipper UART pins (GPIO header):

Pin 13 → TX
Pin 14 → RX
Pin 11 → GND

Connect to ESP32:

Flipper TX → ESP RX
Flipper RX → ESP TX
GND → GND

Baud rate:

115200
📡 ESP32 Firmware (UART responder)

This runs on your ESP32 and listens for commands.

Writing

#include <WiFi.h>

void setup() {
Serial.begin(115200);
}

void loop() {

if (Serial.available()) {

String cmd = Serial.readStringUntil('\n');

cmd.trim();

if (cmd == "SCAN") {

  int n = WiFi.scanNetworks();

  for (int i = 0; i < n; i++) {

    Serial.print("NETWORK:");
    Serial.print(WiFi.SSID(i));
    Serial.print(":");
    Serial.println(WiFi.RSSI(i));

  }

  Serial.println("DONE");
}

}
}

📟 What Happens When You Run It

1️⃣ Start the app on Flipper
2️⃣ Screen shows:

ESP UART Tool
Press OK to scan

3️⃣ Press OK

Flipper sends:

SCAN

4️⃣ ESP32 replies:

NETWORK:HomeWiFi:-52
NETWORK:CoffeeShop:-70
NETWORK:Office:-65
DONE
🚀 Next Steps (where it gets cool)

Once this works, we can add:

Flipper side

scrolling network list

select network

send commands back

ESP side

channel monitor

packet counts

device detection

signal graphs
