*Quick links :*
[Home](/GettingStarted.md) - [Watson Visual](/WatsonVisual.md)

# Programming ESP32 Camera 
This turtorial is a quick guide to setup and get started with ESP-32 Camera microcontroller board. 

## Parts used for this tutorial

- ESP32-CAM with OV2640 camera module
- Female to female jumpers (6 of them)
- The ESP32-CAM doesn’t come with a USB connector, so you need an FTDI programmer to upload code through the U0R and U0T pins. 

You can get the kit here: https://www.amazon.co.uk/KeeYees-ESP32-CAM-Bluetooth-Dual-core-Development/dp/B07S83X9NM

## Integrated Development Environment

This tutorial uses the latest version Arduino IDE to develope the code and load it to the ESP32 CAM. If you're using Mac OS like me, then go here to download the IDE: https://www.arduino.cc/en/guide/macOSX

Once you have the IDE installed, go to `Tools >> Board: >> Board Manager ...` on the menu bar, then search for `ESP32` and install it

![ESP32](images/board-manager.png)

## Load the CameraWebServer example code

Assuming you've had the parts wired up together and connected them to your computer with the Arduino IDE opened. Now open the example code by accessing the menu item following this screenshot

![ESP32](images/web-cam-server.png)

Change the content of `CameraWebServer.ino` file to make it look like this (Keep the `#define CAMERA_MODEL_AI_THINKER` definition and Enter your wifi SSID and Password in)

```C
#include "esp_camera.h"
#include <WiFi.h>

#define CAMERA_MODEL_AI_THINKER

#include "camera_pins.h"

const char* ssid = "your-wifi-ssid";
const char* password = "your-wifi-password";

void startCameraServer();

void setup() {
  Serial.begin(115200);
  Serial.setDebugOutput(true);
  Serial.println();

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;
  //init with high specs to pre-allocate larger buffers
  if(psramFound()){
    config.frame_size = FRAMESIZE_UXGA;
    config.jpeg_quality = 10;
    config.fb_count = 2;
  } else {
    config.frame_size = FRAMESIZE_SVGA;
    config.jpeg_quality = 12;
    config.fb_count = 1;
  }

#if defined(CAMERA_MODEL_ESP_EYE)
  pinMode(13, INPUT_PULLUP);
  pinMode(14, INPUT_PULLUP);
#endif

  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  sensor_t * s = esp_camera_sensor_get();
  //initial sensors are flipped vertically and colors are a bit saturated
  if (s->id.PID == OV3660_PID) {
    s->set_vflip(s, 1);//flip it back
    s->set_brightness(s, 1);//up the blightness just a bit
    s->set_saturation(s, -2);//lower the saturation
  }
  //drop down frame size for higher initial frame rate
  s->set_framesize(s, FRAMESIZE_QVGA);

#if defined(CAMERA_MODEL_M5STACK_WIDE)
  s->set_vflip(s, 1);
  s->set_hmirror(s, 1);
#endif

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");

  startCameraServer();

  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
}

void loop() {
  // put your main code here, to run repeatedly:
  delay(10000);
}
```

Be noticed that in the `void setup()` function, we set `Serial.begin(115200);`, so make sure you setup the `Upload speed` configuration with same value. For example:

![Speed](images/speed.png)


## Connect the parts

Here is the how you wire the ESP32-CAM to the FTDI programmer part

![Parts](images/parts1.png)

(Source: Internet)

Notice that the GPIO 0 pin needs to be connected to GND in order for the code to be uploaded. 

## Upload the code

Before uploading the code, make sure the port that the ESP32-CAM is connected to is selected. Something like this:

![Port](images/port.png)

The partiion option

![Partition](images/partition.png)

and the right Board is selected, like this:

![Board](images/upload1.png)

Press the Reset button on the ESP32-CAM then upload the code to the ESP32-CAM, either using the `Upload` button on the IDE, or `Sketch >> Upload`

Wait until the code uploading done, then disconnect the GPIO 0 from the GND

![Done](images/upload-done.png)

Now open the serial monitor window `Tools >> Serials Monitor` and press the Reset button on the ESP32 board again

You will see something like this that tells you the Video server is ready

![Web](images/monitor.png)

Now you can open a browser on you computer (which is on the same network with the ESP32-CAM board) to access the link, in this case, `http://192.168.1.252/`and start streaming the video

The ESP32-CAM board has the built-in Face detection/recognition capability which is a good foundation to start building your solution. 



