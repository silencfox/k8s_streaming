# ☁️ Cloudflare Tunnel --- Guía Completa (Linux ARM64 / Raspberry Pi)

Esta guía describe cómo instalar **cloudflared**, crear un **Cloudflare
Tunnel**, y configurar rutas tanto **por red** como **por
aplicaciones**, ideal para entornos K3s, Istio, Traefik o servidores
locales.

## 📌 Requisitos Previos

-   Raspberry Pi / Linux ARM64\
-   Cloudflare con dominio configurado\
-   Acceso a la cuenta para autenticación (`cloudflared login`)

# 1. 📥 Instalación de Cloudflared

``` bash
sudo mkdir -p /usr/local/bin
sudo wget -O /usr/local/bin/cloudflared   https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64
sudo chmod +x /usr/local/bin/cloudflared
```

### Verificar versión

``` bash
cloudflared --version
```

### Autenticación con Cloudflare

``` bash
cloudflared login
```

# 2. 🚀 Crear un túnel

``` bash
cloudflared tunnel create nombre-mi-tunnel
```

# 3. 📁 Preparar credenciales del túnel

``` bash
sudo mkdir -p /etc/cloudflared
sudo cp ~/.cloudflared/*.json /etc/cloudflared/
```

# 4. 🌐 Configurar túnel por Red completa (Warp Routing)

### Crear ruta IP

``` bash
cloudflared tunnel route ip add 192.168.1.0/24 nombre-mi-tunnel
```

### Editar configuración

``` bash
sudo nano /etc/cloudflared/config.yml
```

#### config.yml --- Enrutamiento por red

``` yaml
########################################################################
############## Configuración por red / dominio  ################
tunnel: <TUNNEL-ID>
credentials-file: /etc/cloudflared/<TUNNEL-ID>.json

warp-routing:
  enabled: true

ingress:
  - hostname: "*.midomionio.com"
    service: http://192.168.1.20
  - service: http_status:404
########################################################################
```

# 5. 🌍 Configurar túnel por Aplicación

### Crear rutas DNS

``` bash
cloudflared tunnel route dns kdvops-pi-tunnel nginx.kdvops.com
cloudflared tunnel route dns kdvops-pi-tunnel argocd.kdvops.com
```

### Editar configuración

``` bash
sudo nano /etc/cloudflared/config.yml
```

#### config.yml --- Enrutamiento por aplicaciones

``` yaml
########################################################################
# Configuración por aplicación / dominio
tunnel: <TUNNEL-ID>
credentials-file: /etc/cloudflared/<TUNNEL-ID>.json

ingress:
  - hostname: nginx.kdvops.com
    service: http://10.0.0.17

  - hostname: argocd.kdvops.com
    service: http://10.0.0.17
########################################################################
```

# 6. 🛠️ Instalar Cloudflared como servicio (Systemd)

``` bash
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

# ✔️ Conclusión

Con esta guía puedes instalar Cloudflare Tunnel, exponer servicios,
crear rutas DNS, publicar redes completas y ejecutar cloudflared como
servicio en Raspberry Pi.
