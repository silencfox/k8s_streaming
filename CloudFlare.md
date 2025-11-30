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

### crear enrutamiento del tunel por red
cloudflared tunnel route ip add 192.168.1.0/24 nombre-mi-tunnel
sudo nano /etc/cloudflared/config.yml
########################################################################
############## Configuración por red / dominio  ################
# ID del túnel
tunnel: <TUNNEL-ID>
credentials-file: /etc/cloudflared/<TUNNEL-ID>.json

warp-routing:
  enabled: true

ingress:
  - hostname: "*.midomionio.com"
    service: http://192.168.1.20  ## Mi servidor o ip de ingress
  - service: http_status:404

#######################################################################



### crear enrutamiento del tunel por App
cloudflared tunnel route dns kdvops-pi-tunnel nginx.kdvops.com
cloudflared tunnel route dns kdvops-pi-tunnel argocd.kdvops.com

sudo nano /etc/cloudflared/config.yml
########################################################################
# Configuración por aplicación / dominio
# ID del túnel
tunnel: <TUNNEL-ID>
credentials-file: /etc/cloudflared/<TUNNEL-ID>.json

# Cada servicio se publica con su propio hostname
ingress:
  # 1. NGINX DEMO (Publicado por Istio / Traefik)
  - hostname: nginx.kdvops.com
    service: http://10.0.0.17    # IP del Istio/Trafik LB o NodePort

  # 2. ArgoCD
  - hostname: argocd.kdvops.com
    service: http://10.0.0.17    # (puede ser el mismo LB si usas Gateway)
    # Si usas Traefik ->  http://traefik.kube-system.svc.cluster.local:80
    # Si usas Istio ->    http://10.0.0.17
#########################################################################


sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared

