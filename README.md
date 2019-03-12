# PiJam Kochi 2019 - MicroPython Notes

##### [www.b45i.me](http://b45i.me)

## Commands used in demo

### Install esptool

`pip install esptool`

### Flashing Micropython on ESP8266 based boards (wemos d1, nodeMCU etc)

`esptool.py --port /dev/ttyUSB0 erase_flash`
`esptool.py --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 esp8266-20190125-v1.10.bin`

### Flashing Micropython on ESP32

`esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash`
`esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800``write_flash -z 0x1000 esp32-20190125-v1.10.bin`

### Connecting to terminal

`picocom /dev/ttyUSB0 -b115200`

## Code Samples

### Setting pin to output mode

```python
import machine
pin = machine.Pin(2, machine.Pin.OUT)
pin.on() # or pin.value(1)
pin.off() # or pin.value(0)
```

### Blinking an led

```python
import time
while True:
    p.value(not p.value())
    time.sleep_ms(500)
```

### Writing to a file

```python
f = open('data.txt', 'w')
f.write('some data')
f.close()
```

### Reading from a file

```python
f = open('data.txt')
f.read()
'some data'
f.close()
```

### Connecting to wifi

```python
import network
sta_if = network.WLAN(network.STA_IF)
ap_if = network.WLAN(network.AP_IF)
sta_if.active()
ap_if.active()
ap_if.ifconfig()
```

### Function to connect to wifi

```python
def connect_to_wifi(ssid, password):
    import network
    sta_if = network.WLAN(network.STA_IF)
    if not sta_if.isconnected():
        print('connecting to network...')
        sta_if.active(True)
        sta_if.connect(ssid, password)
        while not sta_if.isconnected():
            pass
    print('Connected to %s' % ssid)
    print('network config:', sta_if.ifconfig())
```

### NeoPixel (WS281B) example

```python
import machine, neopixel
np = neopixel.NeoPixel(machine.Pin(4), 8)
np[0] = (255, 0, 0) # set to red, full brightness
np[1] = (0, 128, 0) # set to green, half brightness
np[2] = (0, 0, 64)  # set to blue, quarter brightness
np.write()
```

### NeoPixel animation

```python
import time
def demo(np):
    n = np.n

    # cycle
    for i in range(4 * n):
        for j in range(n):
            np[j] = (0, 0, 0)
        np[i % n] = (255, 255, 255)
        np.write()
        time.sleep_ms(25)

    # bounce
    for i in range(4 * n):
        for j in range(n):
            np[j] = (0, 0, 128)
        if (i // n) % 2 == 0:
            np[i % n] = (0, 0, 0)
        else:
            np[n - 1 - (i % n)] = (0, 0, 0)
        np.write()
        time.sleep_ms(60)

    # fade in/out
    for i in range(0, 4 * 256, 8):
        for j in range(n):
            if (i // 256) % 2 == 0:
                val = i & 0xff
            else:
                val = 255 - (i & 0xff)
            np[j] = (val, 0, 0)
        np.write()

    # clear
    for i in range(n):
        np[i] = (0, 0, 0)
    np.write()

demo(np)
```

### Displaying text on OLED Screen

```python
import machine, ssd1306
i2c = machine.I2C(scl=machine.Pin(4), sda=machine.Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)
oled.fill(0)
oled.text('MicroPython on', 0, 0)
oled.show()
```

### Getting news from web

```python
import network
import urequests
import time


# Yup, im sharing my working API key here ;)
URL = 'http://newsapi.org/v2/top-headlines?sources=the-verge&apiKey=374fb9f276c24901a6c74c82f7bc4d68'

def connect_to_wifi(ssid, password):
    sta_if = network.WLAN(network.STA_IF)
    if not sta_if.isconnected():
        print('connecting to network...')
        sta_if.active(True)
        sta_if.connect(ssid, password)
        while not sta_if.isconnected():
            pass
    print('Connected to %s' % ssid)
    print('network config:', sta_if.ifconfig())


def get_news():
  print('Refreshing.....')
  result = urequests.get(URL)
  if(result.status_code == 200):
    for x in result.json()['articles']:
      print(x['title'])
  time.sleep_ms(1000)

connect_to_wifi('EvilCorp', 'helloworld')

while True:
  get_news()
```

### Final Program, fetching news and desplaying it on OLED display

```python
import network
import urequests
import time
import machine
import ssd1306

i2c = machine.I2C(scl=machine.Pin(4), sda=machine.Pin(5))
oled = ssd1306.SSD1306_I2C(128, 64, i2c)
URL = 'http://newsapi.org/v2/top-headlines?sources=the-verge&apiKey=374fb9f276c24901a6c74c82f7bc4d68'

def display_text(s):
  for i in range(len(s)):
    oled.fill(0)
    oled.text(s, -i, 0)
    oled.show()
    time.sleep_ms(50)

  
def connect_to_wifi(ssid, password):
    sta_if = network.WLAN(network.STA_IF)
    if not sta_if.isconnected():
        print('connecting to network...')
        display_text('connecting to network...')
        sta_if.active(True)
        sta_if.connect(ssid, password)
        while not sta_if.isconnected():
            pass
    print('Connected to %s' % ssid)
    display_text('Connected to %s' % ssid)
    print('network config:', sta_if.ifconfig())


def get_news():
  print('Refreshing.....')
  result = urequests.get(URL)
  if(result.status_code == 200):
    for x in result.json()['articles']:
      print(x['title'])
      display_text(x['title'])
      time.sleep_ms(5000)


connect_to_wifi('EvilCorp', 'helloworld')

while True:
  get_news()
```
