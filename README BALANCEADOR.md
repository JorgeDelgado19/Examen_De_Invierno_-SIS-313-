# **Balanceador**
<div align="justify">

**Objetivo dentro del proyecto:** 

El balanceador en este proyecto distribuye el tráfico web entre las tres aplicaciones Node.js (App1, App2 y App3) utilizando el algoritmo Round Robin, garantizando alta disponibilidad y escalabilidad. Actúa como punto único de entrada para los usuarios, mejorando el rendimiento del sistema.

**Pasos realizados:**

## **Configuración de IP en la máquina virtual**

* **Cambio de IP dinámica a estática:**
```bash
sudo nano /etc/netplan/00-installer-config.yaml

Dentro del archivo:

yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: no
      addresses: [192.168.1.110/24]
Aplicar cambios:

bash
sudo netplan apply
Instalación y configuración de Nginx:
Instalación de Nginx:

bash
sudo apt update && sudo apt install nginx -y
Verificación del Estado del Servicio Nginx:

bash
sudo systemctl status nginx
En caso de que no esté en estado running:

bash
sudo systemctl start nginx
En caso de que no esté habilitado:

bash
sudo systemctl enable nginx
Configuración del archivo de balanceador
Configuración del sitio:

bash
sudo nano /etc/nginx/nginx.conf
Dentro del archivo:

nginx
upstream backend_servers {
    server 192.168.1.120:3000; # App1
    server 192.168.1.130:3000; # App2
    server 192.168.1.140:3000; # App3
}

server {
    listen 80;
    server_name balanceador;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
Revisión de sintaxis:

bash
sudo nginx -t
Reinicio del servicio:

bash
sudo systemctl restart nginx
Configuración de seguridad (Hardening)
Cambio de puerto SSH:

bash
sudo nano /etc/ssh/sshd_config
Modificar:

ini
Port 2201
PasswordAuthentication no
PermitRootLogin no
Reiniciar SSH:

bash
sudo systemctl restart sshd
Configurar firewall:

bash
sudo ufw allow 80/tcp
sudo ufw allow 2201/tcp
sudo ufw enable
Pruebas de funcionamiento
Verificar balanceo:

bash
curl http://192.168.1.110
Prueba desde navegador:
Añadir al archivo hosts de tu máquina local:

bash
192.168.1.110 balanceador-sis313
Luego acceder en navegador a:

text
http://balanceador-sis313
</div> ```