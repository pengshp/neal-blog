---
title: "ESP8266控制DHT11的温湿度采集"
date: 2016-11-24T02:47:34+08:00
categories: [IoT]
tags: [ESP8266, Micropython]
---

项目描述：闲来无事，制作了一个用ESP8266控制DHT11采集温度，湿度并在OLED屏显示的系统，使用了MQTT协议上传网络在手机显示，编程语言使用MicroPython.巩固一下所学到的一些知识。

<!-- more -->

### 一、项目准备

#### 1.所需材料

- DHT11 温湿度传感器
- NUDEMCU 开发板
- IIC 接口 OLED 彩屏
- 小灯，电阻，面包板，导线若干

#### 2.开发环境

- MicroPython
- MQTT

### 二、项目制作

#### 1.电路连接图

![2016112455717wenshidu.png](http://7xseex.com1.z0.glb.clouddn.com/2016112455717wenshidu.png)

按照如上电路图连接，其中`NUDEMCU`的`USB`连接电脑`USB`.

#### 2.编写`main.py`文件

```py
from machine import Pin,I2C
from time import sleep_ms
from ubinascii import hexlify
from umqtt.simple import MQTTClient
from dht import DHT11
from ssd1306 import SSD1306_I2C
import machine,json
#---MQTT Sending---
SERVER="iot.eclipse.org"
CLIENT_TD=hexlify(machine.unique_id())
led = Pin(2, Pin.OUT, value=1)
TOPIC1=b"/dht11/tem"
TOPIC2=b"/dht11/hum"
TOPIC3=b"/dht11/led"
def envioMQTT(server=SERVER,topic="/foo",data=None):
    try:
        c=MQTTClient(CLIENT_TD,server)
        c.connect()
        c.publish(topic,data)
        sleep_ms(200)
        c.disconnect()
    except Exception as e:
        pass
state=0
def sub_cb(topic,msg):
    global state
    print((topic,msg))
    if msg == b"on":
        led.value(1)
        state=1
    elif msg == b"off":
        led.value(0)
        state=0
def recepcionMQTT(server=SERVER,topic=TOPIC3):
    c = MQTTClient(CLIENT_TD,server)
    # Subscribed messages will be delivered to this callback
    c.set_callback(sub_cb)
    c.connect()
    c.subscribe(topic)
    try:
        c.wait_msg()
    finally:
        c.disconnect()
#---End MQTT Sending---
#---DHT11---
ds=DHT11(Pin(16))
def read_dht():
    try:
        ds.measure()
        tem=ds.temperature()
        hum=ds.humidity()
        return (tem,hum)
    except Exception as e:
        return (-1,-1)
#---End DHT11---
#---OLED IIC 128x64---
i2c = I2C(sda = Pin(4), scl = Pin(5))
display = SSD1306_I2C(128, 64, i2c)
def displaytem(tem,hum):
    display.fill(0)
    temperatura = 'Tempera: ' + str(tem)[:5] + 'C'
    humedad = 'Humidty: ' + str(hum)[:5] + '%'
    display.text('DHT11,ESP8266',0,0,1)
    display.text(temperatura,2,24,1)
    display.text(humedad,2,34,1)
    display.show()
#---End OLED---
#---Main Program---
sleep_ms(10000)
def main():
    while True:
        (tem,hum) = read_dht()
        displaytem(tem,hum)
        envioMQTT(SERVER,TOPIC1,json.dumps(tem))
        envioMQTT(SERVER,TOPIC2,json.dumps(hum))
        recepcionMQTT()
        sleep_ms(10000)
#---END Main Program---
if __name__ == '__main__':
    main()
```

#### 3.项目描述

- 使用了`iot.eclipse.org`的`MQTT`服务;
- `ESP8266`运行的是`Micropython`的固件，需自行烧写；
- `main.py`文件使用 `Micropython`的`webrepl`服务上传，复位后自动运行；
- 手机端下载一个`MQTT Client` ,连上`iot.eclipse.org`，并且订阅这些消息便可以看到收到的信息。
  效果如下所示：
  ![2016112581569tem_hum.png](http://7xseex.com1.z0.glb.clouddn.com/2016112581569tem_hum.png)

### 三、参考连接

1.webrepl:<http://micropython.org/webrepl/>
2.MQTT Server:<http://mqtt.org/documentation>
3.iot.eclipse:<http://iot.eclipse.org>
4.MicroPython:<http://micropython.org>
