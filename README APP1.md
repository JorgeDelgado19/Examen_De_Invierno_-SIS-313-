
# **App1 (Servidor de Aplicaciones Node.js)**
<div align="justify">

**Objetivo dentro del proyecto:**  
App1 aloja una aplicación Node.js con operaciones CRUD básicas que se conecta a la base de datos MariaDB. Trabaja en conjunto con App2 y App3 bajo el balanceador de carga.

---

## **📌 Configuración Básica**
- **IP:** `192.168.1.120`
- **Puerto SSH:** `2202`
- **Puerto App Node.js:** `3000`

---

## **🛠️ Configuración de IP Estática**
Archivo: `/etc/netplan/00-installer-config.yaml`
```yaml
network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: no
      addresses: [192.168.1.120/24]

Aplicar cambios:

bash
sudo netplan apply
🚀 Instalación de Node.js
bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs npm
Verificar versión:

bash
node -v  # Debe mostrar v18.x
📦 Aplicación CRUD (Node.js + Express)
1. Estructura del proyecto
bash
mkdir ~/crud-api && cd ~/crud-api
npm init -y
npm install express mysql2 body-parser
2. Código principal (app.js)
javascript
const express = require('express');
const mysql = require('mysql2/promise');
const app = express();
app.use(express.json());

// Configuración BD
const dbConfig = {
  host: '192.168.1.150',
  user: 'app_user',
  password: 'PasswordSeguro123',
  database: 'crud_db',
  port: 3306
};

// Rutas
app.get('/', (req, res) => {
  res.json({ message: 'API CRUD App1', status: 'active' });
});

app.get('/items', async (req, res) => {
  try {
    const connection = await mysql.createConnection(dbConfig);
    const [rows] = await connection.query('SELECT * FROM items');
    res.json(rows);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// POST, PUT, DELETE (similares, ver repositorio completo)
app.listen(3000, () => console.log('App1 en puerto 3000'));
⚙️ Configuración como Servicio Systemd
Archivo: /etc/systemd/system/nodeapp.service

ini
[Unit]
Description=Servidor Node.js App1
After=network.target

[Service]
User=app1
WorkingDirectory=/home/app1/crud-api
ExecStart=/usr/bin/node /home/app1/crud-api/app.js
Restart=always

[Install]
WantedBy=multi-user.target
Comandos:

bash
sudo systemctl daemon-reload
sudo systemctl enable --now nodeapp
🔒 Hardening Aplicado
1. SSH Seguro
bash
sudo nano /etc/ssh/sshd_config
ini
Port 2202
PasswordAuthentication no
AllowUsers app1
2. Firewall (UFW)
bash
sudo ufw allow 3000/tcp
sudo ufw allow 2202/tcp
sudo ufw enable
🧪 Pruebas de Funcionamiento
bash
# Localmente
curl http://localhost:3000/items

# Desde el balanceador
curl http://192.168.1.110
🖥️ Interfaz Web (Opcional)
Archivo: ~/crud-api/public/index.html

html
<!DOCTYPE html>
<html>
<head>
    <title>App1 CRUD</title>
</head>
<body>
    <h1>Ítems desde App1 (192.168.1.120)</h1>
    <ul id="items"></ul>
    <script>
        fetch('/items')
            .then(res => res.json())
            .then(data => {
                data.forEach(item => {
                    document.getElementById('items').innerHTML += `
                        <li>${item.name} <button onclick="deleteItem(${item.id})">🗑️</button></li>
                    `;
                });
            });
    </script>
</body>
</html>
</div> ```