
# **Servidor de Base de Datos (MariaDB + RAID 10)**
<div align="justify">

**Objetivo dentro del proyecto:**  
Este servidor aloja la base de datos MariaDB configurada con RAID 10 para tolerancia a fallos, garantizando la disponibilidad de los datos para las aplicaciones Node.js (App1, App2 y App3).

---

## **📌 Configuración Básica**
- **IP:** `192.168.1.150`
- **Puerto SSH:** `2205`
- **Puerto MariaDB:** `3306`

---

## **🛠️ Configuración de IP Estática**
Archivo: `/etc/netplan/00-installer-config.yaml`
```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: no
      addresses: [192.168.1.150/24]

Aplicar cambios:

bash
sudo netplan apply
🗄️ Instalación de MariaDB
bash
sudo apt update && sudo apt install mariadb-server -y
sudo systemctl enable --now mariadb
🔐 Configuración de Seguridad Inicial
bash
sudo mysql_secure_installation
Establecer contraseña para root

Eliminar usuarios anónimos

Deshabilitar login remoto de root

Eliminar bases de prueba

🛡️ Configuración RAID 10
1. Preparación de discos (4 discos virtuales)
Verificar discos disponibles:

bash
lsblk  # Deben aparecer sdb, sdc, sdd, sde
2. Creación del RAID
bash
sudo mdadm --create --verbose /dev/md0 --level=10 --raid-devices=4 /dev/sd{b,c,d,e}
3. Formateo y montaje
bash
sudo mkfs.ext4 /dev/md0
sudo mkdir /mnt/raid10
sudo mount /dev/md0 /mnt/raid10
4. Configuración permanente
Archivo: /etc/fstab

bash
/dev/md0 /mnt/raid10 ext4 defaults,nofail 0 0
📦 Configuración de la Base de Datos
1. Crear usuario y base de datos
Acceder a MariaDB:

bash
sudo mysql -u root -p
Comandos SQL:

sql
CREATE DATABASE crud_db;
CREATE USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'PasswordSeguro123';
GRANT ALL PRIVILEGES ON crud_db.* TO 'app_user'@'192.168.1.%';
FLUSH PRIVILEGES;
2. Crear tabla de ejemplo
sql
USE crud_db;
CREATE TABLE items (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
🔒 Hardening Aplicado
1. SSH Seguro
bash
sudo nano /etc/ssh/sshd_config
ini
Port 2205
PasswordAuthentication no
AllowUsers bdadmin
2. Firewall (UFW)
bash
sudo ufw allow from 192.168.1.120 to any port 3306  # App1
sudo ufw allow from 192.168.1.130 to any port 3306  # App2
sudo ufw allow from 192.168.1.140 to any port 3306  # App3
sudo ufw allow 2205/tcp
sudo ufw enable
3. Configuración MariaDB
Archivo: /etc/mysql/mariadb.conf.d/50-server.cnf

ini
bind-address = 192.168.1.150
🚨 Recuperación de RAID (Prueba de Fallo)
1. Simular fallo en disco
bash
sudo mdadm /dev/md0 --fail /dev/sdb
sudo mdadm --detail /dev/md0  # Verificar estado
2. Reemplazar disco
bash
sudo mdadm /dev/md0 --remove /dev/sdb
sudo mdadm /dev/md0 --add /dev/sdf  # Disco nuevo
3. Monitorear reconstrucción
bash
watch -n 1 cat /proc/mdstat
🧪 Pruebas de Funcionamiento
1. Desde las Apps
bash
# Desde App1:
mysql -h 192.168.1.150 -u app_user -p
2. Consultas directas
sql
USE crud_db;
INSERT INTO items (name) VALUES ('Prueba1');
SELECT * FROM items;
💾 Backup Automático (Opcional)
bash
sudo crontab -e
Añadir línea:

bash
0 3 * * * mysqldump -u app_user -pPasswordSeguro123 crud_db > /mnt/raid10/backup_$(date +\%F).sql
</div> ```