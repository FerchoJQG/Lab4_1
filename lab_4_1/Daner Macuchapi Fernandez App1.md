# 📝 Informe — Aplicación 1

**Universidad San Francisco Xavier de Chuquisaca**  
**Asignatura:** Infraestructura, Plataformas Tecnológicas y Redes (SIS313)  
**Docente:** Ing. Marcelo Quispe Ortega  
**Semestre:** 1/2026  
**Integrante:** Daner Macuchapi  
**Rol:** APLICACIÓN 1  
**IP SSH:** `192.168.100.178` | **IP VLAN 107:** `192.168.107.3`

---

## 1. Configuración de Red

### 1.1 Configuración IP Estática con VLAN 107

Se editó el archivo de configuración de red:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Contenido aplicado:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      optional: true
      addresses:
        - "192.168.100.178/24"
      routes:
        - to: default
          via: 192.168.100.1
      nameservers:
        addresses:
          - 8.8.8.8
  vlans:
    vlan107:
      id: 107
      link: ens18
      addresses:
        - "192.168.107.3/29"
```

```bash
sudo netplan apply
```

### 1.2 Verificación de Interfaces de Red

```bash
ip addr show
```

> ![](ImagenP/f100.png)

---

### 1.3 Verificación de Conectividad con las demás VMs

```bash
ping -c 3 192.168.107.2   # Proxy - Fernando
ping -c 3 192.168.107.4   # App 2 - Melany
ping -c 3 192.168.107.5   # DB - Limbert
```

![](ImagenP/f101.png)

---

## 2. Cambio de Hostname

```bash
sudo hostnamectl set-hostname app1
```

---

## 3. Instalación de Node.js con NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

```bash
\. "$HOME/.nvm/nvm.sh"
```

```bash
nvm install 22
```

### 3.1 Verificación de Versiones

```bash
node -v
npm -v
```

![](ImagenP/f102.png)
![](ImagenP/f103.png)

---

## 4. Instalación de PM2

```bash
npm install pm2@latest -g
```

### 4.1 Verificación de PM2

```bash
pm2 --version
```

![](ImagenP/f104.png)

---

## 5. Clonación de la Aplicación

```bash
mkdir ~/apps && cd ~/apps
```

```bash
git clone https://github.com/marceloquispeortega/api-restful-crud-movies app1_3000
```

```bash
cd ~/apps/app1_3000 && npm install
```
![](ImagenP/f105.png)


---

## 6. Configuración de Variables de Entorno

```bash
cd ~/apps/app1_3000 && cp .env.example .env
```

```bash
nano ~/apps/app1_3000/.env
```

Contenido aplicado:

```env
PORT=3000
DB_HOST=192.168.107.5
DB_USER=usr_movies
DB_PASSWORD=secret
DB_NAME=db_movies
```

---

## 7. Prueba Manual de la Aplicación

```bash
cd ~/apps/app1_3000 && node app.js
```
![](ImagenP/f106.png)
> ```
> Servidor ejecutándose en el puerto 3000
> Conexión a MariaDB exitosa. Pool creado y probado.
> ```

Se detuvo la app con `Ctrl+C` tras verificar el funcionamiento correcto.

---

## 8. Lanzar con PM2

```bash
cd ~/apps/app1_3000 && pm2 start app.js --name app1_3000
```

```bash
pm2 startup
```

Se ejecutó el comando generado por PM2 (copiado y pegado en la terminal).

```bash
pm2 save
```

### 8.1 Verificación del Estado en PM2

```bash
pm2 status
```

![](ImagenP/f107.png)

---

## 9. Instalación de Node Exporter

```bash
sudo apt install prometheus-node-exporter -y
```

```bash
curl http://localhost:9100/metrics | head -20
```

### 9.1 Verificación del Estado de Node Exporter

```bash
sudo systemctl status prometheus-node-exporter
```

![](ImagenP/f108.png)

---

## 10. Pruebas de Funcionamiento

### 10.1 Verificación del Endpoint desde la VM

```bash
curl http://localhost:3000/movies
```

![](ImagenP/f109.png)

### 10.2 Verificación desde la IP de la VLAN

```bash
curl http://192.168.107.3:3000/movies
```

![](ImagenP/f109.png)

### 10.3 Prueba de Failover

Al solicitar la demostración, se detuvo la App 1 para verificar que el proxy redirige todo el tráfico a App 2:

```bash
pm2 stop app1_3000
```

![](ImagenP/f110.png)


```bash
pm2 start app1_3000
```

![](ImagenP/f111.png)

---

## 11. Conclusiones

- Se instaló Node.js v22 mediante NVM y PM2 como gestor de procesos, garantizando que la aplicación se mantenga en ejecución y se reinicie automáticamente ante cualquier fallo.

- Se configuró correctamente el archivo `.env` apuntando a la base de datos centralizada en `192.168.107.5`, logrando una conexión exitosa a MariaDB desde el primer arranque.

- La aplicación responde correctamente en el puerto `3000` y el endpoint `/movies` devuelve los datos esperados desde la base de datos.

- Durante la prueba de failover, al detener App 1, el proxy de Fernando redirigió automáticamente el 100% del tráfico a App 2, confirmando el correcto funcionamiento de la arquitectura de Alta Disponibilidad.