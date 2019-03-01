# hacksoc28.3.19
All the details of my presentation for those that fall behind

Pre-requisites for the workshop(software)

Any Linux distro 

Arduino IDE (https://www.arduino.cc/en/main/software)

Wireshark (sudo apt-get install wireshark)(Ubuntu/Debian based) 

Python

## GR-GSM(This requires an SDR and some time to compile)
```
sudo apt-get update && \
sudo apt-get install -y \
    cmake \
    autoconf \
    libtool \
    pkg-config \
    build-essential \
    python-docutils \
    libcppunit-dev \
    swig \
    doxygen \
    liblog4cpp5-dev \
    python-scipy \
    python-gtk2 \
    gnuradio-dev \
    gr-osmosdr \
    libosmocore-dev   
```
## Download GR-gsm and compile with (https://github.com/ptrkrysik/gr-gsm)
```
git clone https://git.osmocom.org/gr-gsm
cd gr-gsm
mkdir build
cd build
cmake ..
mkdir $HOME/.grc_gnuradio/ $HOME/.gnuradio/
make
sudo make install
sudo ldconfig
```
## Wifi Deauthentication using ESP8266
(I will provide 4 Nodemcu's you can play around which are pre configured ) 
First we need to flash the NodeMcu using this tutorial (easier to follow the link)
https://github.com/spacehuhn/esp8266_deauther/wiki/Installation


## Pentest part 
This pen test will cover tools like

Nmap
ZAP
metasploit

You are more than welcome to download the Vm and run this on your machine 
https://www.vulnhub.com/entry/basic-pentesting-1,216/


Firstly open up Kali linux or any linux machine 

first do an
```
ifconfig
```

find your IP address run an nmap scan on the network 

```
nmap -sn <your ip>
```

lets run an Nmap scan on the network 

```
nmap -p- -sS -Pn -n -vvv -oA nmap-host-ports 192.168.1.5
```



