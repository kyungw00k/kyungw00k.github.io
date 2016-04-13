# Raspberry Pi 활용

(메모해놓고 그냥 생각날때 보기만 했던 내용을 Posting한다.)

## OS 설치

[공식 Raspbian 이미지](http://www.raspbian.org/RaspbianImages)를 내려받거나, 
[Minimal한 이미지를 다운](http://www.linuxsystems.it/2012/06/raspbian-wheezy-armhf-raspberry-pi-minimal-image/) 받아 SD카드에 설치한다.

인터넷 연결공유로 연결한 후, IP 확인

	$ arp -i bridge0 -a

### Update firmware:

	$ wget https://raw.github.com/Hexxeh/rpi-update/master/rpi-update -O /usr/bin/rpi-update
	$ chmod +x /usr/bin/rpi-update
	$ apt-get install git
	$ rpi-update

### Install raspi-config:
	$ wget http://archive.raspberrypi.org/debian/raspberrypi.gpg.key
	$ apt-key add raspberrypi.gpg.key
	$ echo "deb http://archive.raspberrypi.org/debian/ wheezy main" >> /etc/apt/sources.list
	$ apt-get update
	$ apt-get install raspi-config
	$ raspi-config
	

## Locale 설정

	sudo apt-get install locales
	sudo dpkg-reconfigure locales

## 로컬 시간 동기화
	crontab -e
	
아래 내용을 넣으시면 30분 주기로 Sync합니다.
	
	30 * * * * /usr/bin/ntpdate -b -s -u pool.ntp.org

## 로컬 시간 바꾸기
	ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime

## WIFI 연결
### WPA
	$ sudo nano /etc/network/interfaces

아래 내용을 적어준다.

	auto wlan0
	iface wlan0 inet dhcp
	wpa-conf /etc/wpa.conf

/etc/wpa.conf를 만든다.

	$ wpa_passphrase ssid passkey > /etc/wpa.conf

Adapter를 올린다.

	$ sudo ifup wlan0

### WPA2-Enterprise
* [Video: Configuring your Raspberry Pi for WPA/WPA2 in less than 5 minutes](http://www.youtube.com/watch?v=UGuJrHVd8s8)
* [Configuring WPA2 using wpa_supplicant on the Raspberry Pi](http://kerneldriver.wordpress.com/2012/10/21/configuring-wpa2-using-wpa_supplicant-on-the-raspberry-pi/)

## 참고할만 한 곳
* [Raspbian](http://www.raspbian.org/)
* [R-Pi Troubleshooting](http://elinux.org/R-Pi_Troubleshooting)
* [AFP Mount 하기](http://blog.geekliketodd.com/archives/900)

