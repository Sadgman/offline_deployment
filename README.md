## This guide + code allows for VDO.Ninja to run without Internet on a local network

This guide was tested on a Raspberry Pi with a clean RPI OS installed.

It's recommend you install the image with the Raspberry Pi Imager app, and enable SSH and WiFi via that app, if intending to connect those ways.  

### RPI provided image option

I'm providing a RPi image, but WiFi will need to be configured via boot config, Ethernet, or with keyboard/mouse, or however.

[Dwonload it here](https://drive.google.com/file/d/10WtVXUh7yHxWmdSaR95-E_M3pnUNUtvr/view?usp=sharing) ( 2.4-GB zipped )

The image uses the following user/pass combo:
```
username: vdo 
password: ninja
```

It requires an 8-GB uSD card or larger; if it's too big, you'll need to PiShrink it down first using: https://ostechnix.com/pishrink-make-raspberry-pi-images-smaller/
(I may get around to doing that in the future)

Just burn the image following any RPi guide, login in, and get going.  Currently I think v22.10 (beta) is installed, so probably out of date. You can update, but if you do, be sure to run `sed -i 's/\/\/ session\.customWSS = true;/session\.wss = "wss:\/\/"+window\.location\.hostname+":8443";session\.customWSS = true;/' ./vdo.ninja/index.html` from the home user folder after, or manually update the index.html to point to the local wss server.

You can then load the site at https://192.168.XXX.YYY/ or whereever its loaded.  Accept any SSL concerns or what not (see https://github.com/steveseguin/vdo.ninja/blob/develop/install.md#dealing-with-no-ssl-scenarios for more details)

You may need to download and install the cert https://192.168.XXX.YYY/cert.pem if you don't have allowances for self-signed certs. The required cert is located at https://192.168.XXX.YYY/cert.pem for easy download.

### Installing from scratch 

If installing via scratch, the following is a sample script that might get you going. I'd recommend you run each block manually, a bit at a time, to catch any errors or user input requirements along the way.

We will be using ports 443 and 8443, just in case you have those already in use, you may need to configure it yourself then.

```
### If using a Raspberry Pi, and if having issues updating
sudo chmod 777 /etc/resolv.conf
sudo echo "nameserver 1.1.1.1" >> /etc/resolv.conf
sudo chmod 644 /etc/resolv.conf
sudo chattr -V +i /etc/resolv.conf ### lock access
sudo systemctl restart systemd-resolved.service
export GIT_SSL_NO_VERIFY=1

### Update
sudo apt-get update
sudo apt-get upgrade -y

### Install dependencies
sudo apt-get install git -y
sudo apt-get install nodejs -y
sudo apt-get install npm -y
sudo apt-get install vim -y

### Install vdo.ninja 
git clone https://github.com/steveseguin/vdo.ninja

## configure vdo.ninja for local hss server
sed -i 's/\/\/ session\.customWSS = true;/session\.wss = "wss:\/\/"+window\.location\.hostname+":8443";session\.customWSS = true;/' ./vdo.ninja/index.html

### Install websocket server
git clone https://github.com/steveseguin/offline_deployment
mv offline_deployment webserver
cd webserver
sudo npm install express
sudo npm install ws
sudo npm install fs

## Lets create our self-signed certs
openssl req  -nodes -new -x509  -keyout key.pem -out cert.pem
## Just press enter to skip past the questions at the end of the process

## Make it available to the website, so you can download it and install it
cp cert.pem ../vdo.ninja/cert.pem

## create a service and start the server
sudo cp vdoninja.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable vdoninja
sudo systemctl restart vdoninja

## or start our server directly ..
# sudo nodejs server.js

## show IP addresses
ifconfig 

```

You can then load the site at https://192.168.XXX.YYY/ or whereever its loaded.  Accept any SSL concerns or what not (see https://github.com/steveseguin/vdo.ninja/blob/develop/install.md#dealing-with-no-ssl-scenarios for more details)

You may need to download and install the cert https://192.168.XXX.YYY/cert.pem if you don't have allowances for self-signed certs. The required cert is located at https://192.168.XXX.YYY/cert.pem for easy download.
