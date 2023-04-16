## Set up Guide for the server

> This is a Guide for how to setup my Server

## Installing the Operating System

1. Download the Operating System(OS) you want to use for your server, I am gonna use [Debian](https://www.debian.org/releases/stable/), since it a very stable OS.
2. Burn the iso of the OS on a USB-Sticker, for windows you can use [Rufus](https://rufus.ie/en/) , on linux you can use [popsicle](https://github.com/pop-os/popsicle)
3. Start the PC/Laptop you want to use with the USB-Stick and boot with the USB-Stick
4. Follow the Instruction of the, OS Installer

### Sudo Debian

If you have problems with sudo on debian switch to the root account or add sudo privillige to your current account:
1. `su root`, gets you root access
2. `export PATH="$PATH:/sbin:/usr/sbin:usr/local/sbin"`, allows you to use the adduser command
3. adduser **_username_** sudo
4. exit

### Connecting via ssh

To connect to your server via ssh:
1. Install ssh: `sudo apt-get install openssh-server`
2. Check ssh is running: `sudo systemctl status ssh`
3. Find your IP address or your host name
4. Connect to your system: `ssh username@hostname`

### Configure a Static IP

If you want to Access the server via the same IP, its useful to set a static IP address
Additionally the Standard DNS server your ISP can read your internet flow, that why its good to change your DNS:

#### Way 1: Per Command Line 

1. Show your IP devices: `ip -c link show`
2. Backup your internet config: `sudo cp /etc/network/interfaces ~/`
3. Edit your config: ` sudo nano  /etc/network/interfaces`
	1. Enter(example): 
	 ```sh 
	 auto eth0 
	 iface eth0 inet static address 192.168.178.2 
	 netmask 255.255.255.0 
	 gateway 192.168.178.1 
	 dns-nameservers 8.8.8.8 8.8.4.4
	 ``` 
4. Restart networking: `sudo systemctl restart networking`

I had some issues with this so I recommend way 2.

#### Way 2: Network Manager

Via the Gnome Network Manager:
1. Open your Settings and press on Wifi to set static ip for wifi and Network for static ip for ethernet
2. After that press on the settings tab and switch to the IP4-Tab
3. Select as IPV-4 Method manually
4. for Adress give the ip your pc should have e.g. 192.168.178.100, for netmask it should normally be 255.255.255.0 for gateway select the ip from your router, you can get the router ip by going to your router settings webpage(e.g. fritz.box).
5. You can also select a custom dns

### Enable Automatic Security Updates

It is good practise to auomatically install security-updates for a server:
1. install unattended-upgrades: `sudo apt install unattended-upgrades apt-listchanges bsd-mailx
2. Install, to automatically restart: `sudo apt install apt-config-auto-update`
3. if you sue a laptop: `sudo apt install powermgmt-base`
4. Turn on the security updates: `sudo dpkg-reconfigure -plow unattended-upgrades`
5. Make sure its working: `sudo unattended-upgrades --dry-run --debug`
6. Check its status: `systemctl status unattended-upgrades`

### Protect ssh

In order to not get hacked via ssh, its important to protect your ssh connection:
1. Install fail2ban: `sudo apt install fail2ban`
2. Check its active: `systemctl status fail2ban.service`

### Don't sleep when closing Lid

If you have a Laptop you don't want the laptop to sleep when its lid get closed:
1. `sudo nano /etc/systemd/logind.conf`
2. Change: `#HandleLidSwitch=suspend #HandleLidSwitchExternalPower=suspend` to `HandleLidSwitch=ignore HandleLidSwitchExternalPower=ignore` 

### Turning off the Screen

Having the Screen on, is using power, we can reduce the power consumption by disabling the screen:
1. If you have Gnome installed, do it via gnome settings under power


## Setting up the Services

### Install Docker

If we want to set up anything we should do it with docker, so that requirements/dependencies don't mix and we have Improved security.
1. `sudo apt-get update`
2. Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS
```sh
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg
```
3. Add Docker’s official GPG key:
```sh
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
4. Use the following command to set up the repository:
```sh
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
5. Update apt:
 ```sh
sudo apt-get update
```
6. Install docker:
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
7. Verify its working:
```sh
sudo docker run hello-world
```
8. Make it run rootless, replace `$ROOT` by your username:
```sh
 sudo groupadd docker
 sudo gpasswd -a $USER docker
```
9. logout and login again

### Install Docker-Compose

Compose is used to deploy/run compose files, which are config files for a docker container:

1. Installing compose
```sh
curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```
2. giving it execuable permission
```sh
sudo chmod +x /usr/local/bin/docker-compose
```

### Docker Manager

#### Way 1: Yacht

I had the problem with yacht that it didn't correctly depict cpu usage.

Its nice to have a WebUI to manage/see all docker most popular are yacht and portainer:
1. Crate a folder where you will save docker configs,
```sh
cd /home/
mkdir Docker
```
2. You might Need to edit the volume in `yacht_compose.yml` to the correct path(replace your username).
3. To send a file from you local pc to the server use the following command:
```sh
scp yacht_compose.yml username@hostname:/home/lupos/Documents
```
4. Installing Yacht:
```sh
docker-compose -f yacht_compose.yml up -d
```
5. Stopping with: ``docker-compose down``
6. You can Access yacht via the ip adresss and then the port 8000
7. The standard login is: admin@yacht.local, pass

#### Way 2: Portainer

Portainer is a freemium.

Follow [this guide](https://docs.portainer.io/start/install-ce/server/docker/linux)
1. run the following commands:
```sh
docker volume create portainer_data
docker run -d -p 8001:8000 -p 9445:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
2. You can Access the Dashboard via the website ``https://your_ip:9445`

### Nextcloud

Nextcloud is like google-Cloud, I use it to store files, locally in the server and access them:
1. If you use an external HDD to save your files, navigate to it, it should be in `\mnt\` or `\media`(for removable devices)
2. You might Need to edit the volume in `nextcloud_compose.yml`, to the correct path(replace your username), also edit the password, aldo edit the PUID and PGID to the number you get from the command `id $user`
3. Send the file to your server and Install Nextcloud:
```sh
scp nextcloud_compose.yml username@hostname:/home/lupos/Docker

docker-compose -f nextcloud_compose.yml up -d
```
4. You should be able to reach it under the ip and then the port 444 e.g. 
   `http://192.168.178.1:444`
5. If you have problems with uploading files, Increase the size limit  of file upload:
```sh
cd /config/nginx/site-confs
nano default.conf
```
6. When setting u, choose for storage and database: "MySQL/MariaDB" and then enter what is written in the nextcloud ompose file, it should look like this:
![](Screenshot%20from%202023-04-13%2015-46-07.png)

**Not working well for me**

### Interlude

#### Reinstalling Docker

If sth. went wrong and you want to reinstall the container do the following:
1. Stop the container: 
```sh
docker stop $(docker ps -a -q)
```
2. remove the container:
```sh
docker rm $(docker ps -aq)
```

#### Bad Permission external HDD

If you need sudo for everything you do on you external HDD than your permissions are wrongly set:
1. If your HDD is somewhere else mounted replace /media with where your HDD  is mounted: `sudo chmod ugo+wx /media`
2. Alternatively try: `sudo chown user media`
3. You can investigate with `ls -ld` who the owner is and what privilliges you have.

### Media Stack

#### 1. Torrenting

At First we need the stack that does the torrenting it will be behind a VPN.
1. edit the torrent-compose.yml and change the gluetun serveice [enviroment variable](https://github.com/qdm12/gluetun/wiki) so that they fit for your VPN. e.g. Proton-VPN
```
- VPN_SERVICE_PROVIDER=custom
- VPN_TYPE=wireguard
- VPN_ENDPOINT_IP= 169.150.218.85
- VPN_ENDPOINT_PORT=51820
- WIREGUARD_PUBLIC_KEY=THs9F9xVAiGy48Mc0DnGGPFaVqypz4FtBLbn6Jw2BWM=
- WIREGUARD_PRIVATE_KEY= SElY3AP6XLffrbQoDbpVGlYtTDREMdmWZAzC8fJxg3M=
- WIREGUARD_PRESHARED_KEY=
- WIREGUARD_ADDRESSES=10.2.0.2/32
```
Copied from [here](https://account.proton.me/u/0/vpn/WireGuard).

3. Make the folder Structure for your media, [follow this guide.](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/)
4. Copy the file to your server and run the docker change .env if neseccary
```sh
scp torrent_compose.yml username@hostname:/home/lupos/Docker
scp gluetune/.env username@hostname:/home/lupos/Docker

docker-compose -f torrent_compose.yml up -d
```
5. You should be able to reach it under the ip and then the port e.g. 
   `http://192.168.178.1:8085`
	- qbittorent port: 8085
6. The default username/password is `admin/adminadmin`

#### 2. To Watch

1. Edit the volumes like you need
2. Copy and start it
```sh
scp jellyfin_compose.yml username@hostname:/home/lupos/Docker

docker-compose -f jellyfin_compose.yml up -d
```
3. You should be able to reach it under the ip and then the port e.g. 
   `http://192.168.178.1:8085`
	- jellyfin port: 8096
	- sonarr port: 8989
	- radarr port: 7878
	- bazarr port: 6767
	- lidarr port: 8688
	- readarr port: 8799
	- flaresolverr port: 8191
	- jellyseerr port: 5055
	- calibre port: 8083
	- prowlarr port: 9696

#### 3. Setting it up

We now need to change a bunch of setting to set it up properly.

In general follow this [guide](https://zerodya.net/self-host-jellyfin-media-streaming-stack/).
1. First open Jellyfin and set it up
2. Now set up prowplar 
	1. Add [FlareSolverr](https://trash-guides.info/Prowlarr/prowlarr-setup-flaresolverr/) to Prowlarr
	2. Add trackers
	3. Add apps(sonarr & co)
	4. Add folder structure
3. Set up sonarr, radarr, lidarr, readearr
	1. Add download client
	2. Add index prolwar
4. Set up jellyseer
	1. Add Radarr and Sonarr
5. Set up bazarr

### Shell in a box

With shell in a box we can access the shell via the webrowser.
1. install it: `sudo apt install openssl shellinabox`
2. configure it: `sudo nano /etc/default/shellinabox`
3. Start it with: `sudo systemctl restart shellinabox`
4. listens on port **4200**

### Homer

## Accessing it through vpn

## scinitifc

- https://gitlab.com/mildlyparallel/trashcan

## Resources

- [Setting up Nextcloud](https://techsparx.com/software-development/docker/self-hosting/nextcloud.html)
- [PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid)
- [NFS-Server](https://sysadmins.co.za/setup-a-nfs-server-with-docker/)
- [Samba Server](https://github.com/crazy-max/docker-samba)
- [Media Stack 1](https://zerodya.net/self-host-jellyfin-media-streaming-stack/)
- [Media Stack 2](https://gitlab.com/Pistrie/lootarr/-/blob/master/docker-compose.yaml)
- [Arr wiki](https://wiki.servarr.com/)
- [Media Server Tips and tricks](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/)
