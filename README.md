# Setup butterfly camera

Status: In progress

[Orange Pi 3 and LTS – Armbian](https://www.armbian.com/orangepi3-lts/) 

download CLI version

[balenaEtcher - Flash OS images to SD cards & USB drives](https://www.balena.io/etcher#download-etcher)

setup all the basic with orange pi

create root password: aass123

choose default system command shell: bash

user name : cam

pw: aass123

hostname: cam23XXX, XXX refer to the school list

Locale : en_HK

The entire process might take a while (1 hour) to do so

Get update for link for installnation:

```jsx
sudo apt update
```

Get the basic tools that needs for setup the camera an VPN

```jsx
sudo apt install ffmpeg vim curl
```

Get to know the device IP address, if you know then can do ssh on laptop or tablet

```jsx
ifconfig
```

![Untitled](Setup%20butterfly%20camera%2071949e1967df427194ff5f058997cc9d/Untitled.png)

focus on the one from wlan0, inet is the address we need, make sure we all in the same network

Use nmtui to change the device hostname to cam23XXX

```jsx
nmtui
```

![Untitled](Setup%20butterfly%20camera%2071949e1967df427194ff5f058997cc9d/Untitled%201.png)

here can also change the connection to wifi, MAKE SURE if you attempt to change wifi network all the ssid and password is correct, otherwise once disconnect then you need to use usb connect if you fail to connect new wifi

SSH 

```jsx
ssh cam@<IP Address of Pi>
```

tmux

```jsx
tmux a -t 0
```

If need to leave a session in tmux press Ctrl B +D

Upon complete press Ctrl D to leave the session if you in SSH

or type sudo reboot to reboot the device

```jsx
ffmpeg -re -f v4l2 -framerate 25 -video_size 640x480 -i /dev/video0 -preset ultrafast -an -c:v libx264 -r 60 -tune zerolatency -b 900k -vf "drawtext=fontfileDroidSansMono.ttf: text='%{localtime\:%Y/%m/%d  %T}': r=29.976: x=(w-tw)/2: y=h-(2*lh): fontcolor=white" -pix_fmt yuv420p -f flv "rtmp://live.internal.semproject.us/app/$HOSTNAME"
```

```jsx

```

make sure video device is correct , e. g. change video0 to video1

the last name is the destination name e.g.  camera23XXX

- if DNS fails, like can ping ip address but not domain name, then "rtmp://stream.semproject.us/app/$HOSTNAME" change to "rtmp://[112.118.53.194](https://mxtoolbox.com/SuperTool.aspx?action=a%3astream.semproject.us&run=toolpage#)/app/$HOSTNAME"
- "rtmp://112.118.154.223/app/$HOSTNAME"
- "rtmp://pekopekopeko.us.to/app/$HOSTNAME"

119.236.11.168

[Document (semproject.us)](https://semproject.us/embed.html?device=cam23005)

[live.internal.semproject.us](https://live.internal.semproject.us/embed.html?device=cam23019)

---

## VPN set up

at wireguard In koukeila

koukeila password: password

```bash
pivpn -a

#enter the name e.g. cam23008

cat /home/koukeila/configs/<name_here>.conf
```

at SSH in orange pi

```jsx
sudo nano /etc/wireguard/wg0.conf
```

copy and paste the cert gen from wireguard at koukeila, Ctrl X to exit, with Y to save the data

DNS might not able to be resolve at start so change [pekopekopeko.us.to](http://pekopekopeko.us.to) with [112.118.53.194](https://mxtoolbox.com/SuperTool.aspx?action=a%3apekopekopeko.to.us&run=toolpage#)

Activate the VPN in orange pi

```jsx
wg-quick up wg0
```

Make connection to VPN at boot

```bash
sudo systemctl enable wg-quick@wg0
```

---

[https://sourceforge.net/projects/win32diskimager/](https://sourceforge.net/projects/win32diskimager/)

Save a local copy of the template SD card

## If every pi cant connect, while server is not down

very likely in this case is the docker in the server is down

to check the docker

```jsx
Docker ps -a
```

![Untitled](Setup%20butterfly%20camera%2071949e1967df427194ff5f058997cc9d/Untitled%202.png)

as seen from the first line , Status is exited, which means it is down. To reset it go to home/koukeila/Server/overnMediaEngine

then just reconnect all the pi again

```jsx
cd home/koukeila/Server/overnMediaEngine
docker-compose restart

```

---

## Camera auto on

```jsx
sudo nano /etc/systemd/system/autocam.service
```

```jsx
[Unit]
Description=auto run camera
After=network.target

[Service]
ExecStart=/home/cam/autocam.sh
Restart=always
User=root
Group=root
Type=simple
RestartSec=300
[Install]
WantedBy=multi-user.target
```

Ctrl+X, yes to save

```jsx
sudo nano autocam.sh
```

```jsx
#!/bin/sh
ffmpeg -re -f v4l2 -framerate 25 -video_size 640x480 -i /dev/video1 -preset ultrafast -an -c:v libx264 -r 60 -tune zerolatency -b 900k -vf "drawtext=fontfileDroidSansMono.ttf: text='%{localtime\:%Y/%m/%d  %T}': r=29.976: x=(w-tw)/2: y=h-(2*lh): fontcolor=white" -pix_fmt yuv420p -f flv "rtmp://live.internal.semproject.us/app/cam23019"
```

Ctrl+X, yes to save

reload service

```jsx
sudo systemctl daemon-reload
```

change permission

```jsx
sudo chmod 744 /home/cam/autocam.sh
```

```jsx
sudo chmod 644 /etc/systemd/system/autocam.service
```

```jsx
sudo systemctl enable autocam.service
```

start

```jsx
sudo systemctl start autocam.service
```

check status

```jsx
sudo systemctl status autocam.service
```

check logs

```jsx
journalctl -u autocam.service
```

## Data auto on

```jsx
sudo nano /etc/systemd/system/autodata.service
```

```jsx
                                                                                                              [Unit]
Description=auto run data collect
After=network.target

[Service] 0
ExecStart=/home/cam/autodata.sh
Restart=always
User=root
Group=root
Type=simple
RestartSec=300
[Install]
WantedBy=multi-user.target
```

Ctrl+X, yes to save

```jsx
sudo nano autodata.sh
```

```jsx
#!/bin/sh
sudo apt install python3-pip
pip3 install requests
pip3 install pyserial
python3 /home/cam/serialdata.py
```

Ctrl+X, yes to save

reload service

```jsx
sudo systemctl daemon-reload
```

change permission

```jsx
sudo chmod 744 /home/cam/autodata.sh
```

```jsx
sudo chmod 644 /etc/systemd/system/autodata.service
```

```jsx
sudo systemctl enable autodata.service
```

start

```jsx
sudo systemctl start autodata.service
```

check status

```jsx
sudo systemctl status autodata.service
```

check logs

```jsx
journalctl -u autodata.service
```

---

## Reboot every midnight

```jsx
sudo crontab -e
```

The first time you might have to choose your preferred editor

Insert a line like

```jsx
0 4   *   *   *    /sbin/shutdown -r +5
```

at the bottom. Explanation:

```
m      h    dom        mon   dow       command
minute hour dayOfMonth Month dayOfWeek commandToRun
```

---

| Device ID | Location | Device model | Camera | MQTT PCB | remote.it |
| --- | --- | --- | --- | --- | --- |
| cam23001 | 德望小學 |  | cam23001 |  | U |
| cam23006 | 孫方中 |  | cam23006 |  | U |
| cam23008 | 德望小學 |  | cam23008,cam23007 | yes | U |
| cam23009 | 孫方中 |  | cam23009 | yes | A |
| cam23010 | 孫方中 |  | cam23010 |  | A |
| cam23011 | 李宗德 |  | cam23011 |  | A |
| cam23013 | 羅陳楚思 |  | cam23013 | yes | A |
| cam23014 | 羅陳楚思 |  | cam23014, cam23015 |  | A |
| cam23016 | 李宗德 |  | cam23016 | yes | J |
| cam23017 | 李宗德 |  | cam23017 |  | J |
| cam23018 |  |  |  |  |  |
| cam23021 | 祖堯 |  | cam23021 |  | J |
| cam23019 | 祖堯 |  | cam23022,cam23023 | yes | J |

## **Send data through pi to Onenet**

on esp32

[mqttwithdht11data.zip](Setup%20butterfly%20camera%2071949e1967df427194ff5f058997cc9d/mqttwithdht11data.zip)

on pi

check usb serial 

```jsx
sudo screen /dev/ttyUSB0 115200
```

Ctrl A + K to quit

```jsx
sudo nano serialdata.py
```

put these into serialdata.py

```jsx
import serial
import time
import requests
import json

z1baudrate = 115200
z1port = '/dev/ttyUSB0'  # set the correct port before run it

z1serial = serial.Serial(port=z1port, baudrate=z1baudrate)
z1serial.timeout = 5000  # set read timeout
# print z1serial  # debug serial.
print (z1serial.is_open)  # True for opened
if z1serial.is_open:
    while True:
        size = z1serial.inWaiting()
        if size:
            data = z1serial.read(size)
            temp = float(data.split(b'\r\n')[8])
            humi = float(data.split(b'\r\n')[9])
            print (temp)
            print (humi)
            url = 'http://api.onenet.hk.chinamobile.com/devices/161273852/datapoints'
            h = {
                 "api-Key":"qpUN7EXZEcrLpHFT9qP=zxPz=HY="} # apiKey
            d = json.dumps({
            "datastreams": [{
                        "id": "temperature",
                        "datapoints": [{
                                "value": temp
                            }
                        ]
                    },
                    {
                        "id": "humidity",
                        "datapoints": [{
                                "value": humi
                            }
                        ]
                    }
                ]
            })
            r = requests.post(url, data=d, headers=h)
        else:
            print ('no data')
        time.sleep(10)
```

CTRL+X, press Y, Enter

```jsx
sudo apt install python3-pip
```

```jsx
pip3 install requests
```

waiting until finish

open a new tmux

```jsx
pip3 install pyserial
```

```jsx
python3 serialdata.py
```

check serial data:

```jsx
import serial
import time

z1baudrate = 115200
z1port = '/dev/ttyUSB0'  # set the correct port before run it

z1serial = serial.Serial(port=z1port, baudrate=z1baudrate)
z1serial.timeout = 2  # set read timeout
# print z1serial  # debug serial.
print (z1serial.is_open)  # True for opened
if z1serial.is_open:
    while True:
        size = z1serial.inWaiting()
        if size:
            data = z1serial.read(size)
            print (data)
        else:
            print ('no data')
        time.sleep(1)
```

德望：

```jsx
import serial
import time
import requests
import json

z1baudrate = 115200
z1port = '/dev/ttyUSB0'  # set the correct port before run it

z1serial = serial.Serial(port=z1port, baudrate=z1baudrate)
z1serial.timeout = 2  # set read timeout
# print z1serial  # debug serial.
print (z1serial.is_open)  # True for opened
if z1serial.is_open:
    while True:
        size = z1serial.inWaiting()
        if size:
            data = z1serial.read(size)
            if len(data)<15:
                temp = float(data.split(b'\r\n')[0])
                humi = float(data.split(b'\r\n')[1])
                print (temp)
                print (humi)
                url = 'http://api.onenet.hk.chinamobile.com/devices/161142871/datapoints'
                h = {
                     "api-Key":"Yh6qkDV8557tB1n2o0DbjGzqoV4="} # apiKey
                d = json.dumps({
                "datastreams": [{
                            "id": "temperature",
                            "datapoints": [{
                                    "value": temp
                                }
                            ]
                        },
                        {
                            "id": "humidity",
                            "datapoints": [{
                                    "value": humi
                                }
                            ]
                        }
                    ]
                })
                r = requests.post(url, data=d, headers=h)
            else:
                print ('not true data')
        else:
            print ('no data')
        time.sleep(1)
else:
    print ('z1serial not open')
# z1serial.close()  # close z1serial if z1serial is open.
```