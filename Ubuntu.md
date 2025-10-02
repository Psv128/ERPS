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

# Instalación Odoo

Usaremos la aplicación Virtual Box para crear nuestra máquina virtual.

# Instalación Docker

Usaremos la aplicación Virtual Box para crear nuestra máquina virtual.
