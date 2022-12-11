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
sudo npm install express
sudo npm install ws
sudo npm install fs
sudo apt-get install vim -y

### Install websocket server
git clone https://github.com/steveseguin/websocket_server

mv websocket_server webserver

### Install vdo.ninja 
git clone https://github.com/steveseguin/vdo.ninja

## configure vdo.ninja for local hss server
sed -i 's/\/\/ session\.customWSS = true;/session\.wss = "wss:\/\/"+window\.location\.hostname+":8443";session\.customWSS = true;/' ./vdo.ninja/index.html

## Lets create our self-signed certs
cd webserver
openssl req  -nodes -new -x509  -keyout key.pem -out cert.pem

## Make it available to the website, so you can download it and install it
cp cert.pem ../vdo.ninja/cert.pem


## create a service and start the server
sudo cp vdoninja.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable vdoninja
sudo systemctl restart vdoninja

## or start our server directly ..
# sudo nodejs server.js


## You can then load the site at https://192.168.XXX.YYY/ or whereever its loaded.
## You may need to download and install the cert https://192.168.XXX.YYY/cert.pem if you don't have allowances for self-signed certs

