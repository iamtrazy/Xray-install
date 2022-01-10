## Xray + VLESS-TCP-XTLS/VLESS-gRPC-TLS manual installation tutorial

Prepare the software

- [Xshell 7 Free Edition](https://www.xshell.com/zh/free-for-home-school/)
- [WinSCP](https://winscp.net/eng/docs/lang:chs)

Reinstall the system (newbies are not recommended to start [network reinstallation system](https://github.com/bohanyang/debi), go to your VPS website to operate, and reinstall the system to Deian10 or 11)

- Debian 10
- Debian 11
- Ubuntu 18.04
- Ubuntu 20.04

start installation

- Connect your VPS with Xshell 7
- Login as root user
- Please operate in order from step 0
- If you already have an SSL certificate, rename the public key file to fullchain.pem, rename the private key file to privkey.pem, use WinSCP to connect to your VPS, upload them to the /etc/ssl/private/ directory, execute ` chown -R nobody:nogroup /etc/ssl/private/` command, skip step 1

0. Install curl

````
apt update -y && apt install -y curl
````

1. Apply for [Free SSL Certificate](https://github.com/acmesh-official/acme.sh)

- You need to buy a domain name first, then add a subdomain, and point the subdomain to the IP of your VPS. Because DNS resolution takes a little time, it is recommended to wait 5 minutes after setting, and then execute the following commands (execute each command in sequence). You can check if the returned IP is correct by pinging your subdomain. Note: Replace chika.example.com with your subdomain.

<pre>apt install -y socat

curl https://get.acme.sh | sh

alias acme.sh=~/.acme.sh/acme.sh

acme.sh --upgrade --auto-upgrade

acme.sh --set-default-ca --server letsencrypt

acme.sh --issue -d chika.example.com --standalone --keylength ec-384

acme.sh --install-cert -d chika.example.com --ecc \

--fullchain-file /etc/ssl/private/fullchain.pem \

--key-file /etc/ssl/private/privkey.pem

chown -R nobody:nogroup /etc/ssl/private/</pre>

- Backup the applied SSL certificate: use WinSCP to connect to your VPS, enter the /etc/ssl/private/ directory, download the public key file fullchain.pem and the private key file privkey.pem

2. Install [Nginx](http://nginx.org/en/linux_packages.html)

- Debian 10/11
````
apt install -y gnupg2 ca-certificates lsb-release debian-archive-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring. gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/debian `lsb_release -cs` nginx\n" > /etc /apt/sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d /99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/ systemd/system/nginx.service.d/override.conf
````

- Ubuntu 18.04/20.04
````
apt install -y gnupg2 ca-certificates lsb-release ubuntu-keyring && curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor > /usr/share/keyrings/nginx-archive-keyring.gpg && printf "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] https://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx\n" > /etc/apt /sources.list.d/nginx.list && printf "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d/99nginx && apt update -y && apt install -y nginx && mkdir -p /etc/systemd/system/nginx.service.d && printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/ system/nginx.service.d/override.conf
````

3. Install [Xray](https://github.com/XTLS/Xray-core/releases)

````
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @install --version 1.5.2
````

4.1 Download the configuration files of Nginx and Xray (choose one of two)

- [VLESS-TCP-XTLS](https://github.com/chika0801/Xray-examples/tree/main/VLESS-TCP-XTLS) (recommended)

````
curl -Lo /etc/nginx/nginx.conf https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS/nginx.conf && curl -Lo /usr/local/etc/xray /config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-TCP-XTLS/config_server.json
````

- [VLESS-gRPC-TLS](https://github.com/chika0801/Xray-examples/tree/main/VLESS-gRPC-TLS)

````
curl -Lo /etc/nginx/nginx.conf https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-gRPC-TLS/nginx.conf && curl -Lo /usr/local/etc/xray /config.json https://raw.githubusercontent.com/chika0801/Xray-examples/main/VLESS-gRPC-TLS/config_server.json
````

- If the configuration file is changed, you need to restart Nginx and Xray to make it take effect.

4.2 Download [Routing Rules File Enhanced Edition](https://github.com/Loyalsoldier/v2ray-rules-dat)

````
curl -Lo /usr/local/share/xray/geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat && curl -Lo /usr/local/share /xray/geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat
````

5. Restart Nginx and Xray

````
systemctl stop nginx && systemctl stop xray && systemctl start nginx && systemctl start xray
````

6. Check Nginx and Xray status

````
systemctl status nginx && systemctl status xray
````

7. Others

- Xray configuration file path `/usr/local/etc/xray/config.json` Nginx configuration file path `/etc/nginx/nginx.conf` Routing rule file directory `/usr/local/share/xray`

- How to modify the server configuration file: use WinSCP to connect to your VPS, enter the /usr/local/etc/xray/ directory, double-click the config.json file to edit, find "id": "chika", modify and save, then restart Nginx and Xray to make it work.

- The SSL certificate is automatically renewed every 90 days, and port 80 needs to be used when updating, so port 80 is not monitored in the Nginx configuration file. Applying for a free certificate is limited to 5 times per week, and an error will be reported if the number of times is exceeded, [specific restriction rules](https://letsencrypt.org/zh-cn/docs/rate-limits/)

## Windows System Client Configuration Guide

1. Download v2rayN and Xray

[Open link 1](https://github.com/2dust/v2rayN/releases), click "▸ Assets" in the latest version column, find the link named v2rayN.zip and download it.
[Open link 2](https://github.com/XTLS/Xray-core/releases), click "▸ Assets" in the latest version column, find the link named Xray-windows-64.zip and download it.
Unzip the two compressed packages, copy xray.exe to the v2rayN folder, and run v2rayN.exe.

- Click Settings - Parameter Settings - v2rayN Settings, check "Ignore Geo files when updating Core", change "Core Type" to "Xray_core", OK.
- Click Settings - Routing Settings, set the "Domain Resolution Policy"" to "IPIfNonMatch", uncheck "Enable advanced routing functions", change "Domain Name Matching Algorithm" to "mph", click "Basic Functions", click "One-click Import Basic Rules", OK, OK.
- Right click on the v2rayN icon in the lower right corner of the screen and click on "System Proxy - Automatically Configure System Proxy".

2. Add server in v2rayN

<details><summary>VLESS-TCP-XTLS</summary>

Click on "Server - Add [VLESS] Server", fill in as shown below, and fill in your subdomain for the address (eg chika.example.com)

![VLESS-TCP-XTLS](https://user-images.githubusercontent.com/88967758/132801053-cc8b3aee-5da8-45d5-9e23-115f3b766e52.jpg)</details>

<details><summary>VLESS-gRPC-TLS</summary>

Click on "Server - Add [VLESS] Server", fill in as shown below, and fill in your subdomain for the address (eg chika.example.com)

![VLESS-gRPC](https://user-images.githubusercontent.com/88967758/132800221-1e67083c-6d38-4f00-8f24-38ae688f3d09.jpg)</details>

- Click on the newly added server in the server list and press Enter to load the configuration.

3. Click "Check for Updates"
- Click on "Update GeoSite - Download? - Yes".
- Click on "Update GeoIP - Download? - Yes".

## Android system client configuration guide

1. Download [v2rayNG](https://github.com/2dust/v2rayNg/releases) on your computer, such as v2rayNG_1.x.x_arm64-v8a.apk.

2. Download the enhanced version of the routing rules file on your computer, [geoip.dat](https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat) and [geosite.dat] (https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat).

3. Copy v2rayNG_1.x.x_arm64-v8a.apk, geoip.dat, geosite.dat to the phone with a data cable, and install v2rayNG on the phone.

4. Enter v2rayNG, click `≡` - Settings in the upper left corner, check "Enable Local DNS", "Domain Name Policy" to "IPIfNonMatch", "Predefined Rules" to "Bypass LAN and Mainland Addresses".

5. Click `≡` - Geo resource file in the upper left corner, click `+` in the upper right corner, and select geoip.dat and geosite.dat respectively.

6. Click the v2rayN icon in the lower right corner of the computer, select the server you want to use, and click "Share".

7. Click `+` in the upper right corner - scan QR code, scan the QR code on the screen with your mobile phone.

8. Click on the grey `V` letter icon in the lower right corner.

## Precautions

1.[Why VPS access CN domain name and IP is prohibited](https://github.com/XTLS/Xray-core/discussions/593#discussioncomment-845165).

2. If you use other clients, you need to set up the CN domain name and IP direct connection, otherwise it will be blocked on the VPS side.
