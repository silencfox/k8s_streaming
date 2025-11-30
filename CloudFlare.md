###  cloudflare tunel

sudo mkdir -p /usr/local/bin
sudo wget -O /usr/local/bin/cloudflared \
  https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
sudo chmod +x /usr/local/bin/cloudflared

cloudflared --version
cloudflared login

cloudflared tunnel create nombre-mi-tunnel

sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/*.json /etc/cloudflared/
sudo nano /etc/cloudflared/config.yml

Archivo config
###
tunnel: b1a2c3d4-e5f6-4411-abcd-123456789abc
credentials-file: /etc/cloudflared/b1a2c3d4-e5f6-4411-abcd-123456789abc.json

warp-routing:
  enabled: true

ingress:
  - hostname: "*.midomionio.com"
    service: http://192.168.1.20  ## Mi servidor o ip de ingress
  - service: http_status:404

###

sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared


cloudflared tunnel route ip add 192.168.1.0/24 nombre-mi-tunnel


cloudflared tunnel route dns nombre-mi-tunnel nginx.kdvops.com
