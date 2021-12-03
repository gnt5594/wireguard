## Wireguard-Docker Install Documentation



### Creating droplet in Digital Ocean

- Create a DigitalOcean.com account
- Create a DO Ubuntu Droplet
- Select _Ubuntu 20.04_ ; _Basic_ ; _Regular Intel CPU_
- Choose either SSH key or Password
- Choose data center
- No need for optional addons


### Installing Docker in DO Console

- Click the droplet you made
- Select _Access_ and click _Launch Droplet Console_
- A command line should pop up in a new tab
- Install the necessary tools: sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
- Add Docker Key: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
- Add Docker repo (32bit/64bit OS): sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
- Switch to correct repo: apt-cache policy docker-ce
- Install Docker: sudo apt install docker-ce -y
- If you are not _root_ run this to allow using Docker without _sudo_ in the future: sudo usermod -aG docker ${USER}
- Install Docker-Compose: sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- Set Permission: sudo chmod +x /usr/local/bin/docker-compose


### Installing Wireguard in DO Console

- Run these on your server:
mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
- Copy and paste the content below: 
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Hong_Kong
      - SERVERURL=1.2.3.4
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
 - There are several places you need to modify to your own:
 - _TZ_ refers to timezone so choose yours
 - _SERVERURL_ refers to the server IP address. You can find it on your DigitalOcean dashboard
 - _PEERS_ are the number of user-config-files to generate, or the names of user-config-files
 - Hit _CTRL + X, Y, ENTER_ to save and exit the file
 - Start Wireguard by running these:
  cd ~/wireguard/
  docker-compose up -d
 - It starts building the server. After you see _Creating wireguard ... done_


### Configure Phone

- Check your current IP by putting _IPleak.net_ in your browser
- In the terminal put: docker-compose logs -f wireguard
- It may take a couple of minutes
- Open Wireguard VPN app on your phone, click _+, Create from QR code_ 
- Name your tunnel and activate it
- Check _IPleak.net_ to see if your IP changed. 

### Configure Laptop

- Find your .conf file within Wireguard:
  - cd wireguard
  - ls
  - cd config
  - ls
  - cd peer_pc1
  - ls
  - nano peer_pc1.conf
- Check your current IP address by putting _IPleak.net_ in your browser
- Install Wireguard on your laptop
- Click the bottom left plus sign and create and name a new tunnel
- Paste the text you see from the _peer_pc1.conf_ file, it should look something like this:
 [Interface]
Address = 10.0.0.2
PrivateKey = 0N8QfsKlLKLc4mKNMn7mKsFMuHLeQ7iy5CABijDCnV4=
ListenPort = 51820
DNS = 10.0.0.1

[Peer]
PublicKey = ortjhmEgIvN2SlyF5sJrWmUP1xadkCT/UwjRlUxAD3Q=
Endpoint = 137.184.16.27:51820
AllowedIPs = 0.0.0.0/0, ::/0
- Activate your new tunnel
- Check _IPleak.net_ to see if your IP changed. 

### Phone 
![IMG_0301](https://user-images.githubusercontent.com/54722657/144552065-4030a5ac-5c14-4d12-9836-3efda3233dcc.jpeg)

![IMG_0302](https://user-images.githubusercontent.com/54722657/144552079-a1b5d1aa-96d3-4099-b7c2-c0d33fafc5f4.jpeg)

### Laptop

![IMG_0304](https://user-images.githubusercontent.com/54722657/144552095-5c52ff8e-3c78-4930-80e5-ede0f1c44c20.jpeg)
