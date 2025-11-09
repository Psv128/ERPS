# Documentación Instalación Ubuntu Server

Usaremos la aplicación Virtual Box para crear nuestra máquina virtual.

![alt text](https://img.utdstc.com/icon/dd4/a6e/dd4a6e96b050404041e492471fc933e9d2dd5c24a7bf06e2f0a0e6a43b0f4bb5:200)

# Cambio de Idioma Del Teclado a Español

El teclado por defecto está establecido en inglés , por lo tanto al intentar introducir algunos caracteres especiales no se muestran como en el teclado.
Para arreglar esto, debemos introducir estos comandos para cambiar la distribucion.

`sudo dpkg-reconfigure keyboard-configuration`

<img width="1007" height="750" alt="español1" src="https://github.com/user-attachments/assets/b6a79b38-20ad-4ae9-92f9-2c1c6feca9a0" />

Aqui pondremos un teclado generico y seleccionaremos la distribución en Español.

<img width="1009" height="754" alt="Captura de pantalla 2025-10-02 102633" src="https://github.com/user-attachments/assets/381cbd6c-95e2-4a3b-be15-8e0237d893c6" />

Luego cambiar la distribución de las teclas especiales con este comando:

`sudo dpkg-reconfigure locales`

Aquí normalmente seguiriamos con la instalación del Odoo pero al haber comandos muy extensos instalaremos un SSH para poder copiar y pegar comandos desde la terminal anfitriona.

# Instalación SSH

Para la instalación del SSH, emplearemos estos comandos:

`sudo apt update`
`sudo apt install openssh-server`
`sudo systemctl status ssh`
`sudo systemctl enable --now ssh`

Y desde la maquina anfitriona , para poder establecer la conexión usaremos este comando:

`sudo ssh "nombredelamaquinavirtual"@"ipdelamaquinavirtual` O en mi caso `sudo ssh "vboxuser"@"174.32.45.163`

Y a partir de ahora cualquier comando que usemos en la terminal se ejecturará en la maquina virtual.
Nos aseguraremos de que esta bien hecho si el nombre de la terminal cambia el nombre del equipo al de la maquina virtual

## 🧩 Instalación de Odoo

Para instalar Odoo de forma rápida, limpia y sin complicaciones, utilizaremos **Docker**.  
Docker nos permite ejecutar aplicaciones en contenedores aislados, evitando conflictos de dependencias.

---

## 🐳 Instalación de Docker

Primero, actualizamos los paquetes e instalamos dependencias necesarias:

```bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
```

Añadimos la clave GPG oficial de Docker:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Añadimos el repositorio de Docker:

```bash
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]   https://download.docker.com/linux/ubuntu   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Actualizamos e instalamos Docker:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Comprobamos que el servicio Docker está activo:

```bash
sudo systemctl status docker
```

Si aparece **active (running)**, Docker está correctamente instalado.  
Para no tener que usar `sudo` en cada comando:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## ⚙️ Instalación de Docker Compose

Docker Compose nos permitirá definir y ejecutar varios contenedores con un solo comando.

Instalamos Docker Compose (si no viene incluido):

```bash
sudo apt install docker-compose -y
```

Verificamos la versión:

```bash
docker-compose --version
```

---

## 🧱 Despliegue de Odoo con Docker Compose

Creamos una carpeta para los archivos de Odoo:

```bash
mkdir ~/odoo-docker
cd ~/odoo-docker
```

Creamos el archivo `docker-compose.yml`:

```yaml
version: '3.1'
services:
  web:
    image: odoo:17
    depends_on:
      - db
    ports:
      - "8069:8069"
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./addons:/mnt/extra-addons

  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - odoo-db-data:/var/lib/postgresql/data/pgdata

volumes:
  odoo-web-data:
  odoo-db-data:
```

> 📦 Este archivo define dos servicios:  
> - `web`: contenedor de Odoo (puerto **8069**)  
> - `db`: contenedor de PostgreSQL (base de datos de Odoo)

---

## ▶️ Iniciar Odoo

Ejecutamos el siguiente comando para iniciar los contenedores:

```bash
docker-compose up -d
```

Comprobamos que están en ejecución:

```bash
docker ps
```

Ejemplo de salida:

```
CONTAINER ID   IMAGE         COMMAND                  STATUS         PORTS
123abc456def   odoo:17       "/entrypoint.sh odoo"    Up 2 minutes   0.0.0.0:8069->8069/tcp
789ghi012jkl   postgres:15   "docker-entrypoint.s…"   Up 2 minutes   5432/tcp
```

---

## 🌐 Acceder a Odoo desde el Navegador

Desde la máquina anfitriona, abrimos un navegador y escribimos:

```
http://<ip_de_tu_maquina_virtual>:8069
```

Ejemplo:

```
http://174.32.45.163:8069
```

Veremos la pantalla inicial de configuración de Odoo.  
Creamos una base de datos nueva y establecemos una contraseña maestra (por ejemplo, `admin`).

---

## 🚀 Primeros Pasos en Odoo

### 🛍️ 1. Instalar el módulo de Ventas

1. En el panel principal, ir a **Aplicaciones**  
2. Buscar **Ventas (Sales)**  
3. Hacer clic en **Instalar**

---

### 📦 2. Crear un Producto

1. Abrir el módulo **Ventas**  
2. Ir a **Productos → Productos**  
3. Hacer clic en **Crear**  
4. Rellenar los campos principales:
   - **Nombre:** Producto de ejemplo  
   - **Precio de venta:** 100  
   - **Tipo de producto:** Consumible o Servicio  
5. Guardar los cambios

---

### 🧾 3. Crear un Presupuesto

1. En el módulo **Ventas**, seleccionar **Presupuestos → Crear**  
2. Elegir un cliente (puedes crear uno nuevo)  
3. Añadir productos creados anteriormente  
4. Guardar y hacer clic en **Confirmar Pedido**

---

## 🧰 Gestión de Contenedores

Detener los contenedores:

```bash
docker-compose down
```

Reiniciarlos:

```bash
docker-compose up -d
```

Ver los registros en tiempo real:

```bash
docker logs -f <nombre_del_contenedor_odoo>
```

---
